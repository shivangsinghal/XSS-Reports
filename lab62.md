## Title

Stored XSS in "Profile Signature" field — 62.php

## Affected URL

http://kzlabs.in/62.php

## Description

The "Profile Signature" input field in the account/profile section doesn't sanitize input before storing and rendering it. Payload '"><ImG SrC=X OnErrOr=confirm(1)> was entered into the signature field and saved. On page render, the confirm() dialog fires — meaning the raw HTML gets stored and reflected back unencoded wherever the signature is displayed (e.g., profile page, comments, posts that show the signature).
This is stored XSS — the payload persists in the database and executes for anyone who views a page where the signature is rendered, not just the account owner.

## Steps to Reproduce

1. Create an account at http://kzlabs.in/62.php and go to the profile section.
2. In the "Profile Signature" field, enter: '"><ImG SrC=X OnErrOr=confirm(1)>
3. Save the profile.
4. View the profile page (or wherever the signature renders) — a confirm() dialog pops with 1, confirming stored execution.

## Proof of Concept

<img width="1917" height="745" alt="image" src="https://github.com/user-attachments/assets/927eb068-b48e-4aa8-84cf-c46cb195c337" />



## Impact
Since signatures are commonly shown across multiple pages (profile view, posts, comments, forum threads), this payload could fire for any user who views content tied to this account — not just the owner. This could lead to session/cookie theft, account takeover, or wider compromise if other users or admins encounter the signature while browsing.

## Remediation

1. HTML-encode the signature value everywhere it's rendered (", ', <, >), not just at input time.
2. If basic formatting is intended, use an allow-list HTML sanitizer restricting to safe tags only (e.g., <b>, <i>, links) — block <img>, <script>, and all on* event handlers.
3. Add a CSP to restrict inline event handler execution as a backup layer.
4. Set HttpOnly + Secure on session cookies to limit damage if this fires in another user's session.
