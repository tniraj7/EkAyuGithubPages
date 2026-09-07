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
5. Merge this `docs` folder into the repository's default branch, normally `main`.
6. Push the default branch to GitHub.

## Enable GitHub Pages

1. Open the `codecat15/EkAyu` repository on GitHub.
2. Select **Settings**.
3. In the left sidebar, select **Pages** under **Code and automation**.
4. Under **Build and deployment**, set **Source** to **Deploy from a branch**.
5. Select the repository's default branch, normally `main`.
6. Select the `/docs` folder.
7. Select **Save**.
8. Wait for the Pages deployment to finish. GitHub may take several minutes to publish the first
   deployment.

GitHub Pages publishing from a private repository requires an eligible GitHub plan. If the Pages
settings do not allow this private repository to publish, create a separate public repository named
`ekayu-site`, upload the contents of this folder to that repository's root, and publish from
`main` and `/(root)` instead.

## Verify the website

For publishing from the `EkAyu` repository's `/docs` folder, the expected URLs are:

- Landing page: `https://codecat15.github.io/EkAyu/`
- Privacy Policy URL: `https://codecat15.github.io/EkAyu/privacy-policy.html`
- Support URL: `https://codecat15.github.io/EkAyu/support.html`

Use the URL shown in **Settings → Pages** as authoritative. If a separate `ekayu-site` repository
is used, the paths will begin with `https://codecat15.github.io/ekayu-site/` instead.

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

## Keep the privacy-policy copies synchronized

EkAyu keeps an offline copy of the privacy policy inside the application. Whenever the policy is
changed, update both files:

- Public website: `docs/privacy-policy.html`
- In-app copy: `EkAyu/00. Documentation/privacy-policy.html`

The two copies should have the same effective date and policy text. If the bundled policy changes,
include that change in the next application build submitted to Apple.

## Updating the site later

1. Edit the applicable HTML file in `docs`.
2. If the privacy policy changes, make the same change to the in-app copy and update the effective
   date in both files.
3. Commit and push the changes to the configured Pages branch.
4. Wait for the GitHub Pages deployment to complete.
5. Recheck the public URLs in a private browser window.

Keep the filenames stable after entering the URLs in App Store Connect so the submitted links do
not break.
