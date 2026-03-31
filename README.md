# Secure Communication Guide (GPG)

This guide explains how to use GPG to authenticate messages over an untrusted channel. Every message you send should be signed, and every message you receive should be verified before you act on it.

---

## Phase 1: Setup (Every Student Does This Once)

### Step 1 — Install GPG

| OS | Command |
|---|---|
| Windows | Download [Gpg4win](https://gpg4win.org) |
| macOS | `brew install gnupg` |
| Linux | `sudo apt install gnupg` or `sudo dnf install gnupg2` |

---

### Step 2 — Generate Your Key Pair

Run this in your terminal:

```bash
gpg --full-generate-key
```

When prompted:
- **Key type:** RSA and RSA (default)
- **Key size:** 4096
- **Expiration:** 0 (does not expire, fine for this exercise)
- **Name:** Use your real name so classmates can identify you
- **Email:** Use your university email
- **Passphrase:** Choose a strong one — this protects your private key

---

### Step 3 — Export Your Public Key

```bash
gpg --armor --export your@email.com > pubkey.asc
```

This creates a file called `pubkey.asc` — this is your **public key**. It is safe to share with everyone.

---

### Step 4 — Publish to GitHub

1. Add `pubkey.asc` to the root of your public repo
2. Commit and push

```bash
git add pubkey.asc
git commit -m "Add GPG public key"
git push
```

Your public key will then be available at a trusted, unforgeable URL:

```
https://raw.githubusercontent.com/yourusername/repo/main/pubkey.asc
```

---

## Phase 2: Importing Classmates' Keys (Do This For Each Classmate)

### Step 5 — Download and Import a Classmate's Key

```bash
# Download their key from GitHub
curl -o alice_pubkey.asc https://raw.githubusercontent.com/alice/repo/main/pubkey.asc

# Import it into your GPG keyring
gpg --import alice_pubkey.asc
```

Repeat this for every classmate. You only need to do it once per person.

---

### Step 6 — Verify the Key Fingerprint (Critical!)

After importing, confirm the key is legitimate by checking its fingerprint:

```bash
gpg --fingerprint alice@university.edu
```

This outputs something like:

```
pub   rsa4096 2024-01-01
      A1B2 C3D4 E5F6 7890 ABCD  1234 5678 EF01 2345 6789
uid   Alice Smith <alice@university.edu>
```

Cross-check this fingerprint against a hash Alice posts in her GitHub README. If the fingerprints match, you can be sure the key is genuine and was not tampered with.

---

## Phase 3: Sending Authenticated Messages

### Step 7 — Sign a Message

When you want to send a message in the chat, first sign it:

```bash
echo "Don't click the login link, it's a phishing page." | gpg --clearsign
```

This produces an output like:

```
-----BEGIN PGP SIGNED MESSAGE-----
Hash: SHA512

Don't click the login link, it's a phishing page.
-----BEGIN PGP SIGNATURE-----

iQGzBAABCgAdFiEE....[signature data]....
-----END PGP SIGNATURE-----
```

Paste this entire block into the chat — message and signature together.

---

## Phase 4: Receiving and Verifying Messages

### Step 8 — Verify a Message From Chat

When you receive a signed message, copy the entire PGP block into a file (e.g., `msg.txt`) and run:

```bash
gpg --verify msg.txt
```

GPG will tell you exactly what happened:

✅ **Good signature:**
```
gpg: Good signature from "Alice Smith <alice@university.edu>"
```
The message is genuine. Trust it.

❌ **Bad or missing signature:**
```
gpg: BAD signature from "Alice Smith <alice@university.edu>"
```
The message was forged or tampered with. **Ignore it entirely.**

---

## Optional Phase 5: Encrypting Messages (Hide Content Too)

If you also want to prevent the message from being read in transit, encrypt it for a specific recipient:

```bash
echo "Meet at repo branch 'fallback-plan'" | gpg --clearsign | gpg --encrypt --armor -r bob@university.edu
```

Only Bob can decrypt it with his private key:

```bash
gpg --decrypt message.asc
```

---

## Quick Reference Card

| Task | Command |
|---|---|
| Generate keys | `gpg --full-generate-key` |
| Export public key | `gpg --armor --export you@email.com > pubkey.asc` |
| Import someone's key | `gpg --import their_pubkey.asc` |
| Check a key's fingerprint | `gpg --fingerprint their@email.com` |
| Sign a message | `echo "message" \| gpg --clearsign` |
| Verify a message | `gpg --verify msg.txt` |
| Encrypt for someone | `gpg --encrypt --armor -r them@email.com` |
| Decrypt a message | `gpg --decrypt message.asc` |

---

## Golden Rules

1. **Never trust an unsigned message** — even if it looks like it's from a classmate.
2. **Never share your private key** — not even with teammates.
3. **Always verify the fingerprint** when importing a new key.
4. **If in doubt, don't act** on a message until it's verified.
