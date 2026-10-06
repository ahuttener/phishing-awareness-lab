# 01 — Setup

Everything runs locally on Windows. No admin rights needed.

## 1. Get Gophish

- Release: **v0.12.1**, asset `gophish-v0.12.1-windows-64bit.zip`
- Official source: https://github.com/gophish/gophish/releases/tag/v0.12.1

Download, then **verify the download** before extracting (good habit, and it
shows up well in a portfolio):

```powershell
# from the folder where you saved the zip
Get-FileHash .\gophish-v0.12.1-windows-64bit.zip -Algorithm SHA256
```

Compare the hash against the one published on the release page. If the Windows
"mark of the web" blocked it: right-click the zip → Properties → **Unblock** →
then extract into this repo folder.

```powershell
Expand-Archive .\gophish-v0.12.1-windows-64bit.zip -DestinationPath . -Force
```

> The binary and `gophish.db` are in `.gitignore` — they never get committed.

## 2. Lock the config to localhost

Open `config.json` (created next to `gophish.exe`) and confirm both servers are
bound to loopback only:

```json
{
  "admin_server": { "listen_url": "127.0.0.1:3333", "use_tls": true },
  "phish_server": { "listen_url": "127.0.0.1:80",   "use_tls": false }
}
```

`127.0.0.1` (not `0.0.0.0`) means nothing on your network or the internet can
reach it. This is the single most important safety setting.

## 3. First run

```powershell
.\gophish.exe
```

- The console prints a one-time password: `...username admin and the password XXXX`.
  **Copy it.** Leave this window running — it *is* the server.
- Open https://127.0.0.1:3333 → accept the self-signed certificate warning
  (expected for a local cert) → log in as `admin` → it forces a password change.

## 4. Local mail catcher (recommended)

So that **no mail ever leaves the machine** while you learn the mechanics:

- Download MailHog (Windows): https://github.com/mailhog/MailHog/releases
  → `MailHog_windows_amd64.exe`
- Run it in a second PowerShell window:

```powershell
.\MailHog_windows_amd64.exe
```

- SMTP endpoint it exposes: `127.0.0.1:1025`
- Web inbox to read what was "sent": http://127.0.0.1:8025

You now have a closed loop: Gophish sends → MailHog catches → you read it in a
browser. Move to [`02-run-a-campaign.md`](02-run-a-campaign.md).
