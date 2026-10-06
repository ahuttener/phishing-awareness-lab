# Sample findings — defensive write-up

> **Template.** This is a worked example you fill in / adjust after your own run.
> It's deliberately concrete because *this* — the analysis, not the campaign — is
> what a reviewer reads. Replace the bracketed bits with your real numbers.

## 1. Campaign at a glance

| Metric | Result |
|---|---|
| Campaign | `awareness-test-01` |
| Sending mode | MailHog (local) → then Gmail SMTP for the real-mail analysis |
| Target | 1 recipient — my own mailbox |
| Lure | "Confirm your account within 24 hours" (urgency + authority) |
| Funnel | Sent → Opened `[hh:mm]` → Clicked `[hh:mm]` → landed on reveal page |

## 2. The spoofing surface

The lure's `From:` rendered as **`Keymate Support`** — a friendly display name.
The actual sending address behind it was `[support@keymate-test.local]`, which
does **not** match the brand's real domain.

**Lesson:** users read the *name*; authentication checks the *address*. That gap
is the entire social-engineering surface, and no amount of filtering fixes the
human — but email authentication can stop the *domain* from being impersonated.

## 3. Authentication results (from the real-mail run)

From the received message → *Show original* → `Authentication-Results`:

```
spf=[pass/fail]     smtp.mailfrom=[...]
dkim=[pass/fail]    header.i=@[...]
dmarc=[pass/fail]   (p=[none/quarantine/reject]) header.from=[...]
```

Reading:
- **SPF** — [did the sending IP match the domain's allowed senders? why?]
- **DKIM** — [was the message signed by the domain and intact?]
- **DMARC** — [did SPF/DKIM align with the visible From: domain, and what policy
  did the domain publish?]

## 4. Controls mapped to what I saw

| What the lab showed | Control that addresses it |
|---|---|
| Lookalike display name | user training + client showing the real address |
| `From:` domain spoofable | **DMARC `p=reject`** on that domain |
| Link to a cloned login | URL rewriting / safe-links, blocklists |
| Credentials would be submitted | phishing-resistant MFA (passkeys / FIDO2) |
| Open-tracking pixel fired | image proxying (Gmail does this by default) |

## 5. Real-world tie-in — DMARC posture across three live domains

This lab was run alongside a real DMARC review of three domains that were moved
behind Cloudflare. Their posture after the review:

| Domain | Sends legitimate mail *as itself*? | DMARC policy | Reasoning |
|---|---|---|---|
| `mykeymate.com` | yes (transactional mail) | `p=quarantine` | real senders pass SPF+DKIM; failures go to spam |
| `radarrider.com` | yes | `p=none` → **candidate to raise** | monitor first, confirm all legit mail aligns, then harden |
| `br-deals.com`  | **no** — the site mails as a *different* domain | `p=none` → **safe to go straight to `p=reject`** | nothing legitimate sends as this domain, so a reject policy harms no real mail and refuses every forgery |

**Key insight worth stating in an interview:** the easiest DMARC win is a domain
that *never* sends legitimate mail. You can set `p=reject` immediately — zero risk
to real mail, and every spoof of that `From:` is refused at the receiver's
gateway. That turns the "delivered" spoof from section 2 into a hard bounce.

## 6. If I were defending this at scale

Detections I'd build (mail logs / SIEM):
- inbound mail where `dmarc=fail` **and** `From:` is a brand/executive domain;
- newly-registered **lookalike domains** of our own brands;
- a spike of clicks from one recipient population to a single new external host.
