# App configurations

EkAyu has two shared schemes with independent app identities.

| Scheme | Build configuration | Bundle identifier | Home-screen name | Intended use |
| --- | --- | --- | --- | --- |
| `EkAyu STG` | `Staging` | `com.codecat15.ekayu.stg` | EkAyu STG | Side-loading, debugging, and Xcode Cloud tests |
| `EkAyu PROD` | `Release` | `com.codecat15.ekayu` | EkAyu | App Store archive and production verification |

The bundle identifier defines an iOS app sandbox. Consequently, STG has separate `UserDefaults`, documents, and application-support storage from PROD, and can coexist with the App Store app on a physical device.

## Which scheme should I choose?

Choose **EkAyu STG** when side-loading, debugging, or running Xcode Cloud tests. It installs as `EkAyu STG` with the bundle identifier `com.codecat15.ekayu.stg`, so it can coexist with the App Store version and cannot access that app's data.

The STG scheme allows the app target to participate in the Archive action so Xcode Cloud can discover the staging app as a product. The intended Cloud workflows use Test actions, not Archive actions.

Choose **EkAyu PROD** only for release/archive work and production verification. Do not side-load it onto a device that has the App Store installation, because both use the production bundle identifier.

The Xcode project deliberately has one app target, `EkAyu`, which contains the shared source code. The STG and PROD schemes select different build configurations for that same target; they are not duplicate app targets. `EkAyuTests` and `EkAyuUITests` are test-only targets and are not apps to install.

## Device setup

Select **EkAyu STG** in Xcode before running on a physical device. Automatic signing will need an App ID for `com.codecat15.ekayu.stg` in the configured Apple Developer team.
