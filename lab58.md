## Title

Reflected XSS in URL path segment (username) — 58.php

## Affected URL

https://kzlabs.in/58.php/account/MemeKing99'"><script>alert(1)</script>/messages

## Description

The username segment of the URL path (/account/{username}/messages) is reflected back into the page without encoding. Payload '"><script>alert(1)</script> closes out the current attribute/tag context and injects a live script tag that executes on load. Unlike the earlier findings, this one's injected via the path itself rather than a query parameter, suggesting the app pulls the username straight from the URL path and reflects it somewhere (likely a page header, breadcrumb, or "viewing profile: X" type element) without sanitizing it.

## Steps to Reproduce

1. Visit: https://kzlabs.in/58.php/account/MemeKing99'"><script>alert(1)</script>/messages
2. Alert box pops with 1.
3. View source — the quote/bracket sequence breaks out of the original attribute (likely where the username is displayed), and the raw <script> tag runs directly after it.

## Proof of Concept

<img width="1915" height="575" alt="image" src="https://github.com/user-attachments/assets/494f0817-b4f4-4ba8-a1be-651ea863500d" />


## Impact
Anyone who clicks a crafted profile link like this gets arbitrary JS executed in their session on kzlabs.in. Since this is under /account/, it's likely reachable while a user is logged in — meaning cookie theft, account takeover, or session hijacking are realistic outcomes. Could also be used to impersonate the "MemeKing99" account context to phish other users viewing shared links.

## Remediation

1. HTML-encode the username value before reflecting it anywhere in the page (", ', <, >).
2. Validate the username path segment against expected format (alphanumeric only, length limit) before processing — reject anything with special characters at the routing/input layer.
3. Add a CSP to restrict inline script execution as a backup layer.
4. Set HttpOnly + Secure on session cookies.
