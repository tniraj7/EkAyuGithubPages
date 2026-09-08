# EkAyu GitHub Pages

This folder contains the public App Store privacy and support website for EkAyu.

The Privacy Policy URL and Support URL are separate App Store Connect fields. They use the same
GitHub Pages website but point to different HTML pages.

## Files

- `index.html` — landing page linking to privacy and support.
- `privacy-policy.html` — public privacy policy for App Store Connect.
- `support.html` — public support page with contact information and FAQs.
- `README.md` — deployment and maintenance instructions; it is not part of the user-facing site.

## Before publishing

1. Confirm that `privacy-policy.html` accurately describes the current released version of EkAyu.
2. Confirm that the bundle identifier is `com.codecat15.ekayu`.
3. Confirm that `ekayuapp@gmail.com` is monitored and can receive support messages.
4. Confirm that no user data, medical records, credentials, or secrets have been added to this
   folder.
5. Copy the changed files from this `docs` folder into the site repository described below.
6. Commit and push that repository.

## Where the site is actually published

This `docs` folder is the source of truth for the page *content*, but it is **not** what GitHub
serves. `codecat15/EkAyu` is private and does not publish Pages; `https://codecat15.github.io/EkAyu/`
returns 404.

The live site is served from a separate public repository, **`tniraj7/EkAyuGithubPages`**, published
from `main` and `/(root)`. The URLs registered in App Store Connect, and the URLs the app opens in
Settings, point there.

Editing a file in this folder therefore changes nothing that Apple or a user can see. To publish,
copy the changed file into `tniraj7/EkAyuGithubPages` at the repository root, then commit and push
that repository and wait for its Pages deployment to finish.

## Verify the website

The published EkAyu website URLs are:

- Landing page: `https://tniraj7.github.io/EkAyuGithubPages/index.html`
- Privacy Policy URL: `https://tniraj7.github.io/EkAyuGithubPages/privacy-policy.html`
- Support URL: `https://tniraj7.github.io/EkAyuGithubPages/support.html`

Use the URL shown in **Settings → Pages** as authoritative if the Pages configuration changes.

After GitHub reports a successful deployment:

1. Open all three URLs in a private or incognito browser window.
2. Confirm that none of the pages returns a 404 response or requires a GitHub login.
3. Test the links between the landing, privacy, and support pages.
4. Test the support email link.
5. Check the pages on a phone in both light and dark appearance.

## Add the URLs to App Store Connect

1. Open **App Store Connect → My Apps → EkAyu**.
2. In the app's privacy information, enter the published `privacy-policy.html` address as the
   **Privacy Policy URL**.
3. In the iOS app-version information, enter the published `support.html` address as the
   **Support URL**.
4. Save the changes.
5. Open both saved links from App Store Connect to confirm that they work without authentication.

## Keep the bundled copies synchronized

EkAyu ships offline copies of both pages inside the application. Settings loads the hosted page
first and falls back to the bundled copy when the site is unreachable, so a stale bundled copy is a
stale privacy policy shown to a real user. Whenever a page changes, update both files:

| Page | Public website | In-app copy |
| --- | --- | --- |
| Privacy policy | `docs/privacy-policy.html` | `EkAyu/00. Documentation/privacy-policy.html` |
| Support | `docs/support.html` | `EkAyu/00. Documentation/support.html` |

Each pair should have the same effective date and text. The files are bundled flat at the root of
the app bundle, so their relative links to each other keep working offline. If a bundled copy
changes, include that change in the next application build submitted to Apple.

## Updating the site later

1. Edit the applicable HTML file in `docs`.
2. Make the same change to the matching in-app copy and update the effective date in both files.
3. Commit and push this repository.
4. Copy the changed file into `tniraj7/EkAyuGithubPages`, then commit and push that repository.
5. Wait for the GitHub Pages deployment to complete.
6. Recheck the public URLs in a private browser window.

Keep the filenames stable after entering the URLs in App Store Connect so the submitted links do
not break.
