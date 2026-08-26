# Passkey Security Playbook

A field guide to what passkeys actually protect against, where they still fail,
and how to deploy them without rebuilding the same weaknesses one layer up.

## The stance

**"Passkeys solve phishing" is too broad.**

What passkeys solve is *credential* phishing: a lookalike site can no longer
collect a reusable secret, because there isn't one. The private key never
leaves the authenticator, and WebAuthn assertions are bound to the real origin,
so an adversary-in-the-middle proxy on a lookalike domain gets nothing it can
replay.

That is a genuine protocol-level win, and it matters right now: in August 2026
the Mirage2FA AiTM kit was reported hitting roughly 4,500 US/EU organizations
by abusing Microsoft 365 login flows and MFA-prompt bombing, and WhatsApp added
phishing-resistant passkeys within days of that story. Every AiTM kit success
is an argument for passkeys.

But "unphishable credentials" is not "unbreakable accounts". Attackers move to
the next weakest link, and for passkey deployments those links are known:

- **Synced-key ecosystems** — most consumer passkeys live in a cloud-synced
  store (iCloud Keychain, Google Password Manager). The key material can now be
  recovered by compromising the sync account or an old trusted device.
- **Device compromise** — a passkey proves who you are, not that the device is
  clean. Malware on an unlocked machine can ride the authenticated session or
  drive the authenticator itself.
- **Recovery flows** — attackers skip the ceremony entirely and attack account
  recovery: helpdesk social engineering, email/SMS verification codes.
- **MFA bypass around the ceremony** — deployments that keep OTP/push
  fallbacks run on weakest-factor-wins, and AiTM kits increasingly don't need
  your credentials at all; they need your session after you authenticate.

This repo walks through each of those surfaces and ends with a mitigations
checklist for organizations rolling out passkeys in 2026.

## Where passkeys genuinely help

| Property | Why it kills credential phishing |
|---|---|
| Origin binding | The browser signs and verifies the RP ID. An assertion minted on `login-m365.example.com` is invalid on `login.microsoftonline.com` — an AiTM relay has nothing to forward. |
| No shared secret | The server holds a public key. A breached database leaks nothing phishable or replayable. |
| Hardware-bound keys | With device-bound credentials (TPM, secure enclave, security keys like YubiKey), the private key cannot be exported wholesale. |
| Nothing to spray or stuff | Credential stuffing and password spraying have no target surface. |
| Usability | Signing in is faster than typing a password plus a code, which is why consumer platforms ship them by default now. |

The honest summary: passkeys remove the *front door* that phishing kits were
built to walk through. Everything after this point is about the other doors.

## Sources

- The Hacker News, August 2026: [New passkey attacks can recover synced private keys / bypass phishing-resistant MFA](https://thehackernews.com/2026/08/new-passkey-attacks-can-recover-synced.html)
- Mirage2FA AiTM kit vs ~4,500 US/EU orgs; WhatsApp shipping passkeys days later — reported August 2026 (AI/cyber research digest, 2026-08-26)
