# 02 — Run a campaign

All five pieces are built in the Gophish admin UI (https://127.0.0.1:3333).

## 1. Sending Profile

**Sending Profiles → New Profile.**

### Default: local MailHog (nothing leaves the machine)
| Field | Value |
|---|---|
| Name | `Lab - MailHog` |
| From | `"Keymate Support" <support@keymate-test.local>` |
| Host | `127.0.0.1:1025` |
| Username / Password | *(leave empty)* |
| Ignore Certificate Errors | ✅ |

### Optional: real Gmail (watch it hit a real inbox)
First create a Google **app password** (needs 2-Step Verification on):
https://myaccount.google.com/apppasswords — this is a 16-char password scoped to
this app only; never use your real account password.

| Field | Value |
|---|---|
| Name | `Lab - Gmail` |
| From | `"Keymate Support" <ahuttenerir@gmail.com>` |
| Host | `smtp.gmail.com:587` |
| Username | `ahuttenerir@gmail.com` |
| Password | *(the 16-char app password)* |

Click **Send Test Email → to my own address** and confirm it arrives before saving.

## 2. Landing Page

**Landing Pages → New Page.**
- **Import Site**: paste a login URL you *own* (e.g. your own dev login) so the
  clone is of something you control — never a real third-party bank/login.
- ✅ **Capture Submitted Data** (to see what a victim *would* type).
- **Redirect to**: the reveal page — host `templates/landing-awareness.html`
  somewhere, or paste its contents as a second landing page. The victim always
  ends on "this was a test".

## 3. Email Template

**Email Templates → New Template → Import** the file
[`../templates/email-confirm-account.html`](../templates/email-confirm-account.html).

Key variables Gophish fills in:
- `{{.FirstName}}` — personalisation
- `{{.URL}}` — the tracked link (don't hardcode a URL)
- `{{.TrackingURL}}` / the **Add Tracking Image** box — the open-tracking pixel

## 4. Users & Groups — the scope guardrail

**Users & Groups → New Group.** Add **exactly one row**:

| First Name | Last Name | Email |
|---|---|---|
| Adriano | H | ahuttenerir@gmail.com |

Do not add anyone else. This single-row group is what keeps the lab legal.

## 5. Campaign

**Campaigns → New Campaign.**
- Name: `awareness-test-01`
- Email Template / Landing Page / Sending Profile / Group → the ones above.
- **URL**: `http://127.0.0.1` — the address baked into the link and where the
  landing page is served.
- **Launch Campaign.**

Open the campaign dashboard and watch the funnel update live:
`Email Sent → Email Opened → Clicked Link → Submitted Data`.

Then go analyse it → [`03-defensive-analysis.md`](03-defensive-analysis.md).
