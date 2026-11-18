# Photo Providers

TREK can browse your personal photo library on Immich or Synology Photos and attach selected photos to your journeys and TREK Studio books. TREK never copies the original files. It stores only a reference (provider name + asset ID) and proxies all image streams through its own server, so your provider credentials are never sent to the browser.

> **Admin:** Enable at least one photo provider (Immich or Synology Photos) in **Admin → Addons** — photo provider toggles appear as sub-items under the **Journey** addon. Once a provider is on, its settings card appears in each user's **Settings → Integrations**. If your provider runs on a local or private network, the server must be configured to allow internal network access. See [Admin-Addons](Admin-Addons) and [Internal-Network-Access](Internal-Network-Access).

---

## Supported providers

| Provider | Internal ID |
|----------|-------------|
| Immich | `immich` |
| Synology Photos | `synologyphotos` |

Both providers can be active at the same time.

---

## Configuring a provider

Go to **Settings → Integrations**. Each enabled provider gets its own settings card there, titled with the provider's name — **Immich** and/or **Synology Photos**.

<!-- TODO: screenshot: Photo Providers section in Settings > Integrations -->

### Immich

| Field | Required | Notes |
|-------|----------|-------|
| Server URL | Yes | Full URL of your Immich instance, e.g. `https://immich.example.com` |
| API Key | Yes | Stored encrypted; never returned to the browser after saving |
| Skip SSL certificate verification | No | Switch; lets TREK connect to an Immich server with a self-signed certificate. Off by default |
| Mirror journey photos to Immich on upload | No | Switch; when on, photos you upload in TREK are also pushed to your Immich library |

Enter the full URL of your Immich instance and an Immich API key. The API key is stored encrypted on the TREK server and is never returned to the browser after it is saved.

#### Self-signed certificates

If your Immich server uses a self-signed certificate (for example behind a reverse proxy on your home network), the connection test fails with `fetch failed (self-signed certificate)`. Turn on **Skip SSL certificate verification**, enter the API key again and test or save.

- The setting is per user and only applies to your own Immich connection. Disconnecting Immich, or saving a different server URL without turning it on again, turns it off.
- Only the certificate check is skipped. The connection is still encrypted, but TREK can no longer tell your server from an impostor on the same network, and your API key travels over that connection. Prefer a certificate from a trusted CA where you can.
- Photos you add to a journey are loaded through your connection, so your setting applies when other people view that journey.
- A server on a local or private address still needs internal network access on the TREK server. See [Internal-Network-Access](Internal-Network-Access).

#### Required API key permissions

When generating the API key in Immich (**Account Settings → API Keys**), grant only the scopes TREK actually uses:

| Permission | Why TREK needs it |
|------------|-------------------|
| `user.read` | Verify the API key and identify the connected account |
| `timeline.read` | Browse photos by date |
| `asset.read` | Read photo metadata and search results |
| `asset.view` | Load thumbnails and preview images |
| `album.read` | List owned + shared albums and their contents |
| `asset.download` | Download the assets |
| `asset.upload` | *Only if you enable "Mirror journey photos to Immich on upload"* — push TREK uploads back to your library |

TREK never modifies or deletes anything in Immich, so no `update`, `delete`, or admin scopes are needed.

### Synology Photos

| Field | Required | Notes |
|-------|----------|-------|
| Server URL | Yes | The address of DSM, e.g. `https://your-nas:5001`. TREK adds `/webapi/entry.cgi` to it itself (see [The server URL](#the-server-url)) |
| Username | Yes | Synology account username |
| Password | Yes | Stored encrypted; leave blank to keep the existing password |
| MFA code (if enabled) | No | One-time password for 2FA; only needed on first connection or when re-authenticating |
| Skip SSL certificate verification | No | Switch; disables TLS certificate validation for self-signed certificates |

#### Required DSM account permissions

Synology Photos doesn't use API keys — TREK signs in with a regular DSM user account. To minimize blast radius, create a **dedicated low-privilege DSM user** for TREK rather than reusing your admin account:

- A standard (non-admin) DSM user account is sufficient.
- The account must have access to the **Synology Photos** package (DSM → **Control Panel → User & Group → [user] → Applications**, allow Synology Photos).
- The account must be able to log in to DSM (not disabled, not IP-blocked).
- Network access to DSM (typically port `5000` HTTP / `5001` HTTPS, or your reverse-proxy host).
- 2FA is supported — enter the OTP at first connection; TREK stores the resulting device token so you won't be re-prompted on subsequent saves.
- Read-only access is enough — TREK only lists albums, lists items, runs searches, and fetches thumbnails. It never writes, uploads, or deletes.

#### The server URL

- Enter the address of **DSM** (or of a reverse proxy that forwards DSM), with the port if it is not the default, for example `https://nas.example.com:5001`. TREK signs in through DSM's general API at `<URL>/webapi/entry.cgi`, so the address has to answer there. The plain DSM address does; an address with a path, such as an alias set up for the Photos app, also works on some setups. If the connection test fails, try the plain DSM address first. You can check an address in a browser: `<URL>/webapi/query.cgi?api=SYNO.API.Info&version=1&method=query&query=SYNO.API.Auth` must answer with text that contains `"success":true`.
- DSM's own port is shown under **Control Panel → Login Portal → DSM** (HTTPS port).
- A URL without `http://` or `https://` is treated as `https://`, and a trailing `/` is ignored.
- A NAS on a local or private address needs `ALLOW_INTERNAL_NETWORK=true` on the TREK server; otherwise the connection test fails with an address-not-allowed message. A loopback address such as `localhost` is never reached. See [Internal-Network-Access](Internal-Network-Access).
- **Skip SSL certificate verification** is needed when DSM presents its self-signed certificate. As with Immich, only the certificate check is skipped, so prefer a trusted certificate (for example through DSM's Let's Encrypt support or a reverse proxy) where you can.

#### Sign-in, sessions and two-factor authentication

- TREK signs in to DSM with the username and password, keeps the resulting session ID encrypted, and renews it on its own when DSM reports it as expired (DSM codes 106, 107 and 119). You do not have to sign in again after a DSM restart.
- With **two-factor authentication**, enter the code in **MFA code** and press **Test connection**. DSM hands TREK a trusted-device token (shown in DSM as the device **trek**), which TREK stores encrypted. Later sessions are renewed with that token, so no new code is needed.
- **Saving without a password** keeps the stored one. Saving is refused with *No stored password found* when nothing is stored yet.
- **Changing the server URL or the username** ends the stored session, so you have to test the connection again (with a new MFA code if the account uses two-factor). TREK tells you with the in-app notification **Synology Photos disconnected**; it can be switched off under **Settings → Notifications** as *Synology session cleared*. Saving unchanged account details does not end the session.
- DSM blocks an IP address after repeated failed sign-ins. If TREK reports *Multiple logins blocked from this IP* or *Maximum login attempts reached*, check the block list under **Control Panel → Security → Protection** and allow the TREK server's address.

#### Albums, shared albums and search

- The album list combines your **personal albums**, **albums you have shared** and **albums shared with you**. If one of the three cannot be loaded, TREK still shows the other two.
- For a shared album TREK uses the album's sharing passphrase, so you can attach photos from an album that another DSM user shared with you. The photos still count as yours when you add them: an entry keeps only a reference, and the photo is loaded through the library of the person who added it.
- Photos are searched by date (the day of an entry, the trip period or a date range); the result shows thumbnails and previews that TREK streams through its own server.
- Capture time and GPS position of a photo come from Synology itself. Photos that were attached while TREK read them from the neighbouring item (issue #3019) are checked once against Synology at the first start after that fix, so journey order and map pins come out right; a photo Synology cannot confirm keeps what it had. This runs in the background and needs no action.

---

## Testing the connection

Each provider card has a **Test connection** button next to **Save**. Clicking it sends your current field values to the server and attempts to authenticate with the provider. The card's badge then reads **Connected** (otherwise **Not connected**); any error message from the provider is shown if it fails.

For Synology, a successful test stores a session token so the OTP code is not required again on subsequent saves (as long as the URL and username remain the same).

---

## Troubleshooting Synology Photos

| Message | What it usually means |
|---------|-----------------------|
| *Synology API request failed with status 405* (or *Method not allowed*) | An HTTP 405 means the server or proxy that answered does not accept that request method. When it appears while connecting Synology Photos, check which step shows it: the **Test connection** call goes to your NAS, while **Save** and loading albums go to TREK itself, so a reverse proxy in front of TREK that blocks some methods (for example `PUT`) can cause it. If it comes from the NAS side, a reverse proxy that does not pass `POST` requests to `/webapi/`, or `http://` on an HTTPS port (the redirect turns the POST into a GET), are possible causes. |
| *Failed to connect to Synology API* | Wrong URL or port, the NAS is off or unreachable from the TREK server, or the certificate is rejected. Try the **Skip SSL certificate verification** switch for a self-signed certificate. |
| An address-not-allowed message | The NAS is on a private address and `ALLOW_INTERNAL_NETWORK` is off, or the address is loopback. See [Internal-Network-Access](Internal-Network-Access). |
| *Invalid credentials.* (400) | Wrong username or password. |
| *Session expired or account disabled.* (401) / *No permission to use this account.* (402) | The DSM account is disabled or not allowed to sign in. |
| *Two-factor authentication code required.* (403) | Enter the current code in **MFA code** and test again. |
| *Two-factor authentication failed.* (404) | The code was wrong or expired; use the next one. |
| *Two-factor authentication is enforced for this account.* (406) | The account has to use two-factor on DSM's side; enter a code. |
| *Maximum login attempts reached.* (407) / *Multiple logins blocked from this IP.* (107) | DSM has blocked the TREK server's address. Unblock it in DSM and wait a moment. |
| *Password expired.* (408, 409) / *Password must be changed before login.* (410) | Change the password in DSM, then enter it in TREK again. |
| *Insufficient privilege.* (105) | The account may not use the **Synology Photos** package. Allow it under **Control Panel → User & Group**. |

Passwords and sessions are encrypted with the server's encryption key; see [Encryption-Key-Rotation](Encryption-Key-Rotation) if you rotate it.

---

## Multiple providers

You can configure both Immich and Synology simultaneously. TREK offers the photos of all connected providers side by side in the journey photo pickers.

---

## After setup

Once a provider is connected, you can browse and attach its photos in your journeys and in TREK Studio. Providers are used there only; the trip planner itself has no photo tab. For files such as tickets and PDFs, see [Documents-and-Files](Documents-and-Files).

In the [Journey Journal](Journey-Journal) the same libraries are used in three places:

- the **External photos** tab of the entry editor, which searches the entry's day and puts photos taken near the entry first (see [External photos](Journey-Journal#external-photos));
- the journey gallery's photo picker, with **Select all** over a day, the **Trip Period**, a **Date Range** or **All Photos**;
- TREK Studio, where the **Content** section offers **From Immich** and **From Synology Photos** next to **Upload photos** (see [TREK Studio](Journey-Journal#trek-studio)).

---

## See also

- [Admin-Addons](Admin-Addons)
- [Internal-Network-Access](Internal-Network-Access)
- [TREK-Studio](TREK-Studio)
