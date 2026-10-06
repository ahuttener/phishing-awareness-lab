# Email authentication checklist — SPF / DKIM / DMARC

A reference for the defensive write-up. These three records decide whether a
spoofed `From:` reaches the inbox.

## The one-line version

| Record | Question it answers | Lives at |
|---|---|---|
| **SPF**  | Is this *sending IP* allowed to send for the domain? | `TXT` on the domain (`v=spf1 ...`) |
| **DKIM** | Is the message *signed* by the domain and unmodified? | `TXT` at `selector._domainkey.domain` |
| **DMARC**| If SPF/DKIM don't *align* with the visible `From:`, what should the receiver do? | `TXT` at `_dmarc.domain` |

## How to read each one

```bash
# SPF — who may send
dig +short TXT example.com         # look for the v=spf1 line

# DKIM — is a signing key published for the selector
dig +short TXT hostingermail-a._domainkey.example.com

# DMARC — the policy
dig +short TXT _dmarc.example.com   # v=DMARC1; p=none|quarantine|reject; ...
```

On Windows:

```powershell
Resolve-DnsName -Type TXT _dmarc.example.com
```

## DMARC policy ladder

- `p=none` — **monitor only.** Nothing is blocked; you just collect reports.
  Safe first step while you confirm legitimate mail passes.
- `p=quarantine` — failing mail goes to **spam**.
- `p=reject` — failing mail is **refused at the gateway**. Strongest protection
  against someone forging your `From:` domain.

Move up the ladder only once you're sure every *legitimate* source of mail for
the domain passes SPF **and** DKIM — otherwise you send your own mail to spam.

## Worked example (from a real rollout)

Three domains were moved behind Cloudflare and their DMARC posture reviewed:

| Domain | Sends real mail as itself? | DMARC decision |
|---|---|---|
| `mykeymate.com` | yes | `p=quarantine` (already in place) |
| `radarrider.com` | yes | `p=none` → candidate to raise |
| `br-deals.com` | **no** (site mails as another domain) | safe to go straight to `p=reject` |

The takeaway worth writing up: a domain that **never** sends legitimate mail is
the easiest win — set `p=reject` and no real mail can be harmed, while every
forgery of it gets refused.
