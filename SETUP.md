# JC AppVanta: private update server setup

This is a one-time setup, and everything in it is free. After it's done, publishing an update just means uploading one file to GitHub.

**How it works:** AppVanta 1.2.0 and newer check `https://updates.jcappvanta.com/api/update` each time they open. That address is a small function running on Netlify. It only answers requests that carry AppVanta's hidden app code, and everything else gets "Not found". When a request has the right code, the function reads the latest release from your **private** GitHub repo and hands back a download link that expires in about 5 minutes. Nobody can browse your releases or download them from a browser.

---

## 1. Create the private repo

1. On GitHub, click **New repository**.
2. Name it `jc-appvanta-updates`, choose **Private**, then click **Create repository**.
3. Click **uploading an existing file** and drag in everything from this folder: `netlify.toml`, `package.json`, the `netlify` folder and the `public` folder. Then click **Commit changes**.

## 2. Make a read-only GitHub token

1. On GitHub, go to your profile picture → **Settings** → **Developer settings** → **Personal access tokens** → **Fine-grained tokens** → **Generate new token**.
2. Fill it in:
   - **Name:** `AppVanta update server`
   - **Expiration:** the longest option allowed (or no expiration). Put a reminder in your calendar a week before it runs out, because updates stop when it expires.
   - **Repository access:** Only select repositories → `jc-appvanta-updates`
   - **Permissions → Repository permissions → Contents:** Read-only
3. Click **Generate token** and copy it right away. GitHub only shows it once.

## 3. Put the function on Netlify

1. In Netlify, go to **Add new project** → **Import an existing project** → **GitHub** and pick `jc-appvanta-updates`. If it isn't listed, click the link to configure the Netlify app on GitHub and give it access to that repo.
2. Leave the build settings as they are (`netlify.toml` handles them) and deploy.
3. Go to **Site configuration → Environment variables** and add these three:

| Key | Value |
|---|---|
| `APPVANTA_APP_SECRET` | the `"secret"` value from `update.config.json` in the AppVanta source zip (copy it exactly) |
| `GITHUB_TOKEN` | the token from step 2 |
| `GITHUB_REPO` | `your-github-username/jc-appvanta-updates` |

4. Go to **Deploys → Trigger deploy → Deploy site** so the function picks up the variables.

## 4. Point `updates.jcappvanta.com` at it

1. In the new site, go to **Domain management → Add a domain** and enter `updates.jcappvanta.com`.
2. If jcappvanta.com uses Netlify DNS, this is set up automatically. If not, add a **CNAME** record at your domain registrar: `updates` → `<your-new-site>.netlify.app`.
3. Wait for HTTPS to show as active (Netlify does this on its own).

**Check it:** open `https://updates.jcappvanta.com/api/update` in a browser. It should say **Not found**, which means it's locked correctly.

---

## Publishing an update (every time)

1. Get the new `JC-AppVanta-Setup-<version>.exe` (send me your updated HTML and I'll build it).
2. In the `jc-appvanta-updates` repo, go to **Releases → Draft a new release**.
3. Fill it in:
   - **Tag:** `v` plus the version, for example `v1.3.0`. It must match the version of the exe you attach.
   - **Title:** anything, for example `JC AppVanta 1.3.0`
   - **Description:** what's new. Users see this text in the update window.
   - **Attach:** drag in `JC-AppVanta-Setup-1.3.0.exe`
4. Click **Publish release**.

The next time someone opens AppVanta, they'll see "JC AppVanta 1.3.0 is ready" with your notes, and **Update now** / **Later** buttons. Update now downloads the new version, installs it and reopens AppVanta. Their projects are kept. They can also check any time from the command palette: **Ctrl+K → Check for updates**.

**To pull a bad release,** delete it or mark it as a pre-release. The server only offers the latest full release.

## Good to know

- **Existing buyers:** copies older than 1.2.0 don't have the update checker. Give those buyers 1.2.0 once, for example by replacing the file in Lemon Squeezy. After that, they update from inside the app.
- **Exported apps:** apps your customers export never contain the update settings or the hidden code.
- **Changing the address or the code:** `update.config.json` in the source holds the update address and the hidden code. If you change either one, rebuild AppVanta, and change `APPVANTA_APP_SECRET` in Netlify to match.
- **Limits:** the hidden code isn't stored as plain text in the exe, but a determined person could still dig it out. If it ever leaks, change the code in `update.config.json` and in Netlify, then ship a new version to buyers the usual way.
