# EkAyu iCloud synchronization behavior

This document defines the product behavior implemented by the Core Data and CloudKit stack. It is intentionally phrased as synchronization rather than versioned backup.

## User choice

- iCloud synchronization is opt-in on each device.
- Enabling synchronization, from Settings or the existing-data offer below, re-adds the loaded local store with CloudKit options, so syncing starts without a relaunch.
- Disabling synchronization re-adds the loaded store without CloudKit options, so syncing stops without a relaunch. The preference is turned off first, so if detaching fails the next launch still opens the store local-only.
- Both modes use the same protected local SQLite store. Disabling synchronization never deletes the local copy.
- While synchronization is disabled, the user can continue to create, update, and delete records. Persistent history remains enabled so those changes can be exported when synchronization is enabled again.
- **Delete Data from iCloud** is a separate destructive action. It disables synchronization, preserves the local SQLite store and attachment cache, and deletes the record zones in EkAyu's private CloudKit container.
- The request is bound to the signed-in iCloud account before it is saved. If the account can't be read, no request is saved and Settings explains why, so a signed-out device is never left with a pending request that locks the sync toggle.
- CloudKit is detached from the running store before any zone is deleted, so this device's mirroring never observes the deletion. If detaching fails, the request stays pending with synchronization off, and the next launch loads the store local-only and retries.
- A failed cloud deletion leaves synchronization off and keeps the request pending for retry. EkAyu never reports completion until CloudKit confirms every zone deletion.
- EkAyu cannot remotely turn off synchronization on another installation. The user must turn it off on every other device before deletion; otherwise an enabled device can upload its retained local records again.
- **Delete All App Data** removes EkAyu's data from this device only. It replaces the local store with an empty one that does not synchronize and turns synchronization off for this device, so no deletion reaches iCloud or other devices, either immediately or when synchronization is turned on again. Turning synchronization back on downloads the iCloud copy.

## Existing iCloud data on a new device

- When EkAyu opens with no local profiles, no sync choice ever made on this device, no pending cloud deletion, and an available iCloud account, it checks the private CloudKit database for EkAyu profiles.
- Only an unarchived profile record in Core Data's mirrored zone counts. Catalog records such as seeded medical options, doctors, and facilities do not, because every syncing device uploads them.
- The check only reads record metadata and the archive flag, so attachments are never downloaded. A missing zone, network failure, or other error shows no offer and the check repeats on a later launch.
- If profiles exist, EkAyu asks whether to turn on iCloud Sync. Accepting turns the setting on and re-adds the loaded local store with CloudKit options, so syncing starts without a relaunch. EkAyu shows sync progress until the first imported profile arrives, and the user can continue to onboarding without waiting. Onboarding still moves to the home screen when imported profiles arrive, unless the user has started adding a profile.
- Declining records sync as off, so the offer does not return and the Settings toggle stays off.
- Because CloudKit can attach during a session, **Delete Data from iCloud** always detaches the running store at the moment of the request, not only when sync was on at launch.

## Synchronization semantics

- Profiles, reports, attachments, doctors, facilities, report types, and profile medical-option catalog changes synchronize through the user's private iCloud database.
- Inserts, updates, and deletions synchronize to other EkAyu installations that use the same iCloud account and have synchronization enabled.
- Synchronization is asynchronous. A successful local save does not mean another device has received the change.
- A deletion is authoritative. EkAyu does not promise recovery of a record that another device edited while offline after that record was deleted.
- An edit saves only the fields, medical values, and attachments the user changed since the editor opened. Changes another device made in the meantime are kept, including attachments it added, and attachments it deleted are not recreated. When both devices change the same field, the later save wins.
- Screens that list profiles and reports reload when changes from another device are imported.

## Availability and failure behavior

- EkAyu checks the device's iCloud account status before accepting a request to enable synchronization.
- A missing, restricted, temporarily unavailable, or indeterminate iCloud account prevents enablement and leaves all health data local.
- Apple does not provide a reliable remaining-storage preflight. EkAyu detects a full iCloud quota only after CloudKit reports an export failure.
- Network, service, and quota failures never delete or roll back the local store. EkAyu retains local changes and allows the system to retry.
- Settings reports requested configuration separately from operational state. A preference alone is never displayed as proof that data is synchronized.

## User-visible states

- **Stored on this device** — synchronization is not requested.
- **Restart required** — attaching or detaching CloudKit failed, so the running store doesn't match the requested mode. The next launch applies the requested mode before the store loads.
- **Setting up iCloud** — CloudKit is preparing the mirrored store.
- **Syncing** — an import or export is in progress.
- **Synced** — the most recent observed CloudKit operation completed successfully and no export is blocked by a full iCloud quota. This is not a point-in-time backup guarantee.
- **Sync paused** — a retryable network or service condition, such as rate limiting or a busy zone, interrupted synchronization; local editing remains available.
- **iCloud account unavailable** — the device cannot currently use the required private iCloud database. Because CloudKit can report a signed-out account as a generic error, EkAyu checks the account after an unexplained failure and whenever the account changes.
- **iCloud storage full** — CloudKit rejected an export for quota. This state remains until a later export succeeds, even if imports succeed in the meantime; local data remains intact.
- **Sync needs attention** — a non-retryable setup, import, or export error occurred.

## Privacy and scope

- Cloud data is stored in the user's private CloudKit database and is available only through the user's iCloud account and EkAyu's CloudKit container.
- Health and identity fields (profile details, medical lists, report titles, notes and dates, attachments and their names, and doctor, facility, report-type, and medical-option names) are encrypted on the device before upload, using keys in the user's iCloud Keychain. Metadata such as IDs, dates of creation, and archive flags stays unencrypted so sync and the existing-data offer can read it.
- Encryption can't be added to a field that already exists in either CloudKit schema environment, and can never be removed after production deployment.
- EkAyu has no separate account system and does not operate an independent backup server.
- Deleting synchronized data is expected to propagate to iCloud and other enabled devices. **Delete All App Data** is the exception: it affects only this device.

## Identity and conflict boundary

- Profiles and reports are user records. EkAyu must not merge them merely because two records have the same display name or title.
- Self, Father, and Mother are one-per-account relationships, but devices that were offline or not syncing can each create one. Sync keeps every such profile and never rewrites a relationship on its own.
- The primary profile is the earliest-created active Self profile, with the lexical UUID string as the tie-breaker. It is derived when profiles are read, so every device shows the same primary without writing anything back.
- The earliest-created holder of Self, Father, or Mother can still be edited. A later duplicate can be saved only after the user picks a different relationship, so new edits never add to the duplication.
- Doctors, facilities, report types, and profile medical options are catalog records. Concurrent creation on offline devices can produce semantically duplicate rows because CloudKit does not support Core Data unique constraints.
- The deterministic reconciliation policy for catalog records is: group by the existing normalized semantic key; prefer a seeded medical option when one exists; otherwise prefer the earliest `createdOn`; use the lexical UUID string as the final tie-breaker; redirect relationships to the winner; and archive losing catalog rows before considering deletion.
- A profile medical option deleted by the user carries a separate deletion marker. When that marker reaches a device, reconciliation archives every row with the same category and normalized name and removes matching profile assignments. Archiving a duplicate does not create a deletion marker. Explicitly adding the option again clears markers on matching rows already present on that device and restores one row.
- Reconciliation runs after imported persistent-history transactions contain catalog changes. The local two-store harness verifies deterministic convergence, but Cloud sync is not ready for production rollout until that behavior is also validated with the real CloudKit service.

## Deployment boundary

- These phases configure the app and local store for CloudKit, but do not mutate the live CloudKit schema.
- Initialize the development schema deliberately from a development build, inspect it in CloudKit Console, run the real-device synchronization matrix, and deploy the verified schema to production before enabling the feature for users.
