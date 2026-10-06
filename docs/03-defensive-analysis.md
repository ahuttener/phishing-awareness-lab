# 03 — Defensive analysis (the part that gets you hired)

Building the campaign is 20% of the value. Explaining *why it worked or failed*,
in defender's terms, is the other 80%. Work through these after a run and write
your findings into the README's "What I learned" section.

## A. Read the received email's authentication results

If you used the Gmail sending mode, open the message in Gmail →
**⋮ → Show original**. Find the `Authentication-Results` header:

```
Authentication-Results: mx.google.com;
       spf=pass (google.com: domain of ...) smtp.mailfrom=...
       dkim=pass header.i=@...
       dmarc=pass (p=NONE ...) header.from=...
```

For each, answer:
- **SPF** — did the sending IP match the domain's allowed senders?
- **DKIM** — was the message cryptographically signed by the domain, intact?
- **DMARC** — did SPF/DKIM *align* with the visible `From:` domain, and what
  **policy** (`p=none/quarantine/reject`) did the domain ask for?

## B. The spoofing lesson

Notice the gap between what the **user sees** (the display name
`"Keymate Support"`) and the **actual sending address**. Users read the name;
authentication checks the address. That gap is the entire social-engineering
surface. A control can't fix the human — but DMARC can stop the *domain* from
being impersonated at all.

## C. Map it to real controls

| Observation in the lab | Control that addresses it |
|---|---|
| Lookalike display name | User training + client showing the real address |
| `From:` domain spoofed, SPF/DKIM fail | **DMARC `p=reject`** on that domain |
| Link to a cloned login | URL rewriting / safe-links, blocklists |
| Credentials submitted | MFA (phishing-resistant: passkeys/FIDO2) |
| Pixel fired = message opened | Image proxying (Gmail does this by default) |

## D. Tie it to a real deployment

This lab pairs with a real DMARC rollout I did across three domains
(`radarrider.com`, `mykeymate.com`, `br-deals.com`) behind Cloudflare. Document
the before/after of a `_dmarc` record and how a spoof attempt is treated under
`p=none` vs `p=reject`:

```bash
# how to read a domain's current DMARC policy
nslookup -type=TXT _dmarc.example.com
# or
dig +short TXT _dmarc.example.com
```

A reject policy means a receiver *refuses* mail that forges that `From:` domain —
turning the spoof in part B from "delivered" into "rejected at the gateway".

## E. Detection angle (bonus)

If you run a mail log or a SIEM, write the detection you'd build:
- alert on inbound mail where `dmarc=fail` and `From:` is a brand/exec domain;
- alert on newly-registered lookalike domains of your own brand;
- alert on a spike of clicks to a single new external host.
