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

## Where passkeys still fail

### 1. Synced-key ecosystems

Most passkeys people use today are not hardware-bound. They are generated once
and then replicated through a cloud sync service so they follow the user across
laptop, tablet, and phone. That trade is deliberate — it is what makes passkeys
survive a lost phone — but it changes the threat model from "steal the key out
of the secure enclave" to "compromise the ecosystem that escrows the key":

- The sync account becomes a high-value target. Its protections (password,
  recovery email, legacy sessions on a decade-old laptop) are now the real
  front door to every synced passkey.
- Old devices linger. A phone in a drawer, still signed into the same cloud
  account, holds or can fetch the same credentials years later.
- August 2026 research covered by The Hacker News described attacks that can
  recover synced private keys — the class of result that turns "the key never
  leaves my device" marketing into "the key leaves your device whenever the
  provider decides it should".

None of this makes synced passkeys bad. It makes them *account-compromise
targets* rather than *phishing targets*, which is a different problem that
password managers already taught us: convenience features concentrate risk.

### 2. Device compromise

A successful WebAuthn ceremony says "a person holding this authenticator was
present". It says nothing about what else is running on that machine:

- Malware running in the user's session does not need to export anything. It
  can wait for an unlocked device, invoke the authenticator at the right
  moment, or simply ride the authenticated cookie jar afterwards.
- On a compromised endpoint, "approve this prompt" is one more dialog the
  attacker's tooling can click. Local malware can also register *new*
  credentials on accounts it controls while the legitimate owner stays enrolled.
- This is why passkey rollouts still need endpoint hygiene, session-lifetime
  controls, and device health signals. A passkey authenticates the human; it
  does not disinfect the browser.

### 3. Recovery flows

Attackers read the docs too: if resetting access takes three steps and the
first two are "call the helpdesk" and "read a code from SMS", the WebAuthn
ceremony protecting step four is decoration. Real-world failure patterns:

- Helpdesk/social-engineering resets — an attacker with enough personal data
  gets a fresh enrollment of their own passkey on the victim's account.
- Email/SMS verification codes used as recovery proof — reintroduces exactly
  the phishable, interceptable factors passkeys removed.
- Silent re-enrollment flows with no notification — victims find out days later.

Account recovery is where strong authentication goes to die. If the recovery
path is weaker than the login path, the recovery path *is* the login path.

### 4. MFA bypass around the ceremony

Two ways deployments leak despite good ceremonies:

- **Weakest-factor-wins fallbacks.** Keep OTP or push-approval as an
  alternative login method and every benefit above is optional. An AiTM kit
  like Mirage2FA does not defeat WebAuthn — it goes around it, abusing M365
  login flows and MFA-prompt bombing until someone approves. Kits like that
  productized prompt fatigue at scale (~4,500 organizations reported hit).
- **Post-auth session theft.** Modern kits increasingly do not need your
  credentials at all: phish the session cookie after a legitimate login and the
  strength of the ceremony that created the session is irrelevant. Passkeys
  protect credential creation, not session lifetime.

The pattern across all four surfaces: passkeys moved the wall, and attackers
walked along it until they found where it ends.

## Mitigations checklist for organizations

*(next section)*

## Sources

- The Hacker News, August 2026: [New passkey attacks can recover synced private keys / bypass phishing-resistant MFA](https://thehackernews.com/2026/08/new-passkey-attacks-can-recover-synced.html)
- Mirage2FA AiTM kit vs ~4,500 US/EU orgs; WhatsApp shipping passkeys days later — reported August 2026 (AI/cyber research digest, 2026-08-26)
