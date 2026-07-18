## Title

Reflected XSS in p parameter — 56.php

## Affected URL

https://kzlabs.in/56.php?p=%27%22%3Eshivay%3Cscript%3Ealert(1)%3C/script%3E

## Description

The p parameter is reflected back into the page without encoding. Payload '"><script>alert(1)</script> closes out the current attribute/tag context and injects a live script tag that runs immediately.

## Steps to Reproduce

1. Visit: https://kzlabs.in/56.php?p=%27%22%3Eshivay%3Cscript%3Ealert(1)%3C/script%3E
2. Alert box pops with 1.
3. View source — the quote/bracket sequence breaks out of the original attribute, and the raw <script> tag executes directly after it.

## Proof of Concept

<img width="1910" height="645" alt="image" src="https://github.com/user-attachments/assets/2186c87f-0230-4a7c-91d1-998739bff005" />



## Impact

Attacker can send a victim this link and run arbitrary JS in their browser under the kzlabs.in origin. Session cookie theft (if not HttpOnly), account takeover, phishing overlays, or redirecting the user to an attacker-controlled page are all possible.

## Remediation

1. HTML-encode the p value before reflecting it into the page (", ', <, >).
2. Validate/sanitize p server-side based on what it's actually meant to hold.
3. Add a CSP to restrict inline script execution as a backup layer.
4. Set HttpOnly + Secure on session cookies.
