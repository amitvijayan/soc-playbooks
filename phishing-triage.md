# Phishing Triage Playbook

**Trigger:** user-reported suspicious email, or mail gateway alert.
**Goal:** determine malicious vs. benign within 15 minutes, preserve evidence either way.
**Write-up:** [Phishing triage: headers, links and safe evidence handling](https://amitvijayan.com/journal/articles/phishing-triage-headers-links-and-safe-evidence-handling.html)

## 1. Do not click anything yet

Open the message in a safe viewer (quarantine console, or export the `.eml`/`.msg`). Work from the raw message, not the rendered preview.

## 2. Headers — verify the sender

- [ ] Check `Return-Path`, `Reply-To`, and `From` — do all three agree? A mismatch is the first red flag.
- [ ] Check `Authentication-Results`: SPF, DKIM, DMARC — `pass` / `fail` / `none` for each.
- [ ] Check `Received` chain: does the sending IP's reverse DNS and ASN match the claimed sender's infrastructure?
- [ ] Look for display-name spoofing (`"CEO Name" <random@gmail.com>`).

**Verdict point:** SPF/DKIM/DMARC all failing + mismatched reply-to = treat as malicious until proven otherwise.

## 3. Links and attachments — detonate safely

- [ ] Hover/extract URLs — expand shorteners first (`curl -sIL` or a URL expander).
- [ ] Compare display text vs. actual href. Punycode / lookalike domains go here.
- [ ] Submit URLs to urlscan.io (public) or your sandbox. Never open in a production browser.
- [ ] Attachments: check extension vs. MIME, detonate in sandbox. `.html`/`.svg` attachments with credential forms are the current fashion.

## 4. Scope — who else got it?

- [ ] Search the mail gateway for the same sender/subject/message-ID across all mailboxes.
- [ ] List every recipient. If anyone clicked: pivot to endpoint — browser history, process execution around the click time.
- [ ] Check for inbox rules created by the account (persistence via mail rules is common after credential theft).

## 5. Evidence handling

- [ ] Preserve the original message (export `.eml`, keep gateway logs).
- [ ] Record: sender, URLs, attachment hashes (SHA256), recipients, clickers.
- [ ] If malicious: block sender/domain/URLs at gateway + proxy, reset credentials of clickers, open incident if there was execution.

## 6. Close out

- [ ] Notify the reporter (people stop reporting when nobody replies).
- [ ] Tag the ticket: `phishing-malicious` or `phishing-benign`.
- [ ] If benign: note why, so the next analyst doesn't re-triage it.
