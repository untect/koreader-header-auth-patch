# KOReader Header Authentication

A lightweight Lua patch for [KOReader](https://github.com/koreader/koreader) that enables authentication against **Pangolin** and **Cloudflare Zero Trust** using headers.

This patch allows you to access OPDS catalogs (like [Calibre-Web Automated](https://github.com/crocodilestick/Calibre-Web-Automated) or [Kavita](https://github.com/Kareadita/Kavita)) as well as other KOReader compatible apps (like [BookOrbit](https://github.com/bookorbit/bookorbit)) that are protected behind Pangolin or Cloudflare without needing a browser login, VPN client, or complex proxy setups on your e-reader.

Credit to [crocodilestick](https://github.com/crocodilestick) and [vicegold](https://github.com/vicegold) for the original [Cloudflare](https://github.com/crocodilestick/koreader-cloudflare-auth-patch)/[Pangolin](https://github.com/vicegold/koreader-pangolin-auth-patch) implementations.

## 🚀 How It Works
KOReader natively supports HTTP/HTTPS but does not support the interactive login flows required by services like Cloudflare or Pangolin.

This script uses **"Monkey Patching"** to hook into the core Lua network libraries (`socket.http` and `ssl.https`) inside KOReader. It intercepts every network request made by the device and automatically injects the required headers before the request leaves the device.

## 🛠️ Prerequisites
1.  A device running **KOReader** (Kindle, Kobo, Android, etc.).
2.  A **Pangolin** instance or **Cloudflare Zero Trust** account protecting your OPDS/BookOrbit server.

## ⚙️ Setup
Before installing the patch, you must generate the credentials with either Pangolin or Cloudflare. These act as the machine-to-machine username/password for your device

### Pangolin

1.  Open your **Pangolin Dashboard**.
2.  Navigate to the **Shareable Links** section under **Access Control**.
3.  Click **Create Shareable Link**.
    * **Resource:** Select your OPDS/BookOrbit resource.
    * **Title:** `KOReader Device` (or similar). _Optional but recommended._
    * **Associate User:** Select which Pangolin user this token has been issued to. _Optional but recommended._
    * **Expiration:** Select `Never expire` (recommended) or configure a custom expiration period.
    * **Persist Session:** Leave off, as the patch will inject the token with every request.
4.  **Copy the "Token ID" and "Token" immediately.** You will not be able to see the token again. 
    * _If not immediately visible, these can be found under `See Access Token Usage`._

### Cloudflare

1.  Open your **Cloudflare Zero Trust Dashboard**.
2.  Navigate to **Access** > **Service Auth**.
3.  Click **Create Service Token**.
    * **Name:** `KOReader Device` (or similar).
    * **Duration:** Set to `Non-expiring` (recommended) or configure a custom expiration period.
4.  **Copy the "Client ID" and "Client Secret" immediately.** You will not be able to see the secret again.
5.  Navigate to **Access** > **Applications** and select your application.
6.  Add a new **Policy** (or edit your existing one):
    * **Action:** `Service Auth` (Recommended) or `Allow`.
    * **Rule:** Select `Service Token` and choose the token you just created.


## 📥 Installation

1.  Download either the `2-pangolin-auth.lua` _**or**_ the `2-cloudflare-auth.lua` file from this repository.
2.  Open the file in a text editor (Notepad++, VS Code, etc.).
3.  Replace the placeholder credentials with your tokens.
    * For Pangolin:
    ```lua
    local P_TOKEN_ID = "<your-token-id-here>"
    local P_TOKEN = "<your-token-here>"
    ```
    * For Cloudflare:
    ```lua
    local CF_ID = "<your-client-id-here>"
    local CF_SECRET = "<your-client-secret-here>"
    ```
4.  Replace the placeholder URL with the domain name you have protected by Pangolin or Cloudflare.
    ```lua
    local TARGET_DOMAIN = "<your-sub.domain.com>"
    ```
5.  Connect your KOReader device to your computer via USB.
6.  Navigate to the KOReader directory (`/koreader/patches/`) on the device.
    * *(Note: If the `patches` folder does not exist, create it).*
7.  Copy your modified `2-pangolin-auth.lua` _**or**_ the `2-cloudflare-auth.lua` into that folder.
8.  **Restart KOReader** (Exit and re-open, or full reboot).

## 🔍 Verification & Troubleshooting
This patch integrates with KOReader's internal logging system. If you are having issues:

1.  Open the `crash.log` file in your KOReader directory.
2.  Search for `Pangolin-Auth` or `CF-Auth`.
3.  You should see success messages like:
    ```text
    Pangolin-Auth: Initializing...
    Pangolin-Auth: ✓✓✓ Hooks installed successfully ✓✓✓
    Pangolin-Auth: ✓ Injected headers for URL: [https://your-opds-url.com/opds](https://your-opds-url.com/opds)
    ```

### Common Issues
* **"Unable to Connect":** Check your ID and Token/Secret for typos. Ensure you have correctly configured Cloudflare access policy includes the token.
* **Boot Loop:** If KOReader crashes on boot, delete the file from the `patches` folder via USB.
* **HTTP Traffic:** The patch currently only injects into HTTPS traffic as that is the default behaviour of both Pangolin and Cloudflare. If your resource is configured for access over HTTP, you will need to enable HTTP injection here:
    ```lua
    local INJECT_HTTP = true
    ```

## ⚠️ Security Considerations
Your **Access Token** is stored in plain text on the device.
* If you lose your device, anyone with USB access could potentially copy the token.
* **Mitigation:** If your device is lost or stolen, simply revoke the Access Token in the Pangolin/Cloudflare Dashboard. This will immediately cut off access without needing to change your server passwords.

## 📄 License
MIT License. Feel free to use, modify, and distribute.
