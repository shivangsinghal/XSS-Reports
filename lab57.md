## Title

Reflected XSS in returnTo parameter — 57.php

## Affected URL

https://kzlabs.in/57.php?returnTo=%27%22%3Eshivay%3Cscript%3Ealert(1)%3C/script%3E

## Description

The returnTo parameter is reflected back into the page without encoding. Payload '"><script>alert(1)</script> closes out the current attribute/tag context and injects a live script tag that executes on load.

## Steps to Reproduce

1. Visit: https://kzlabs.in/57.php?returnTo=%27%22%3Eshivay%3Cscript%3Ealert(1)%3C/script%3E
2. Alert box pops with 1.
3. View source — the quote/bracket sequence breaks out of the original attribute, and the raw <script> tag runs directly after it.

## Proof of Concept

<img width="1906" height="670" alt="image" src="https://github.com/user-attachments/assets/3ddf34ce-fe29-42ae-a25c-8cadbc0accf6" />


## Impact

Since returnTo params are often used in redirect/login flows, this is worth flagging as higher risk — an attacker could combine this with the redirect logic to build a convincing phishing chain, or simply run JS in the victim's session for cookie theft, account takeover, or page defacement if a crafted link is clicked.

## Remediation

1. HTML-encode the returnTo value before reflecting it into the page (", ', <, >).
2. If returnTo is meant to be a URL/path, validate it against an allow-list of expected internal paths rather than reflecting it raw.
3. Add a CSP to restrict inline script execution as a backup layer.
4. Set HttpOnly + Secure on session cookies.
