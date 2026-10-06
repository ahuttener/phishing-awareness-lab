# Phishing Awareness Lab

A self-contained, **ethically-scoped** lab that builds a controlled phishing
campaign with [Gophish](https://getgophish.com/) and then analyses it from the
**defender's** side — email authentication (SPF / DKIM / DMARC), header forensics
and user-facing warning signs.

> **Why this exists.** Phishing is still the number-one initial access vector in
> real breaches. The best way to learn to *defend* against it is to build one in
> a closed environment and dissect exactly why it does — or does not — reach the
> inbox. This repo does that against a single consenting target: **me**.

---

## 🔒 Scope & ethics (read first)

This lab is **only** ever run against the author's own mailbox, on the author's
own machine, with no services exposed to the internet.

- The only recipient is the author's own e-mail address.
- The admin panel and phishing server bind to `127.0.0.1` (localhost) only.
- No real third party is ever targeted. No credentials of any real service are
  harvested — the capture page points at a login the author controls.
- Every simulated message ends on a **reveal page** that explains it was a test.

Running a phishing message against anyone else, even as a prank, without written
authorisation is illegal in most jurisdictions (Ireland: *Criminal Justice
(Offences Relating to Information Systems) Act 2017*; Brazil: *Lei 14.155/2021*).
**Don't.** The skill that gets you hired is doing this *safely and in scope*.

---

## What's in here

```
phishing-awareness-lab/
├── README.md                     ← you are here
├── LICENSE
├── .gitignore                    ← keeps the gophish binary & DB out of git
├── docs/
│   ├── 01-setup.md               ← install Gophish + local mail catcher
│   ├── 02-run-a-campaign.md      ← build template → landing → group → launch
│   └── 03-defensive-analysis.md  ← the part that matters: why it worked/failed
├── templates/
│   ├── email-confirm-account.html   ← the lure (import into Gophish)
│   └── landing-awareness.html       ← the reveal page ("this was a test")
└── analysis/
    └── email-auth-checklist.md      ← SPF/DKIM/DMARC reading guide
```

## Architecture

```
            ┌──────────────────────────────────────────────┐
            │              your machine (localhost)         │
            │                                               │
  Gophish   │   ┌─────────┐   lure email    ┌───────────┐   │
  admin ────┼──▶│ Gophish │────────────────▶│ MailHog   │   │
  :3333     │   │ server  │                 │ :1025     │   │
            │   └────┬────┘                 └─────┬─────┘   │
            │        │ tracked link               │ web UI  │
            │        ▼                             ▼         │
            │   ┌──────────┐                http://127.0.0.1:8025
            │   │ landing/ │  ← you click, it logs the event,
            │   │ reveal   │    then shows "this was a test"
            │   └──────────┘                               │
            └──────────────────────────────────────────────┘
```

Two sending modes are documented:

1. **MailHog (default)** — a local mail catcher. Nothing leaves the machine; you
   read the captured mail in a browser. Zero credentials, zero blast radius.
2. **Real Gmail SMTP** — optional, to watch the lure actually land in an inbox
   and to inspect real SPF/DKIM/DMARC results. Uses a Google *app password*,
   never the account password.

## Quick start

```powershell
# 1. download Gophish (see docs/01-setup.md for the verified link + hash)
# 2. from this folder:
.\gophish.exe
# 3. note the one-time admin password printed in the console
# 4. open https://127.0.0.1:3333  (self-signed cert warning is expected)
```

Then follow [`docs/02-run-a-campaign.md`](docs/02-run-a-campaign.md).

## What I learned (fill this in as you go)

This section is the real portfolio value. After running a campaign, document:

- How the forged `From:` name renders in the client vs. the real sending address.
- What the received message's **Authentication-Results** header says
  (`spf=`, `dkim=`, `dmarc=`) and *why*.
- Whether Gmail flagged it, and which signal did it.
- One concrete control that would have stopped it (e.g. a strict DMARC policy on
  the spoofed domain — see [`analysis/email-auth-checklist.md`](analysis/email-auth-checklist.md)).

---

## License

MIT — see [LICENSE](LICENSE). Educational use only, within the scope above.
