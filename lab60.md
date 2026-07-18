## Title

Stored XSS in "Full Name" field — 60.php (registration/profile)

## Affected URL

http://kzlabs.in/60.php

## Description

The "Full Name" input field on the registration form doesn't sanitize input before storing and later rendering it. Payload '">shivay<ImG SrC=X OnErrOr=confirm(1)> was entered as the full name during signup. After account creation and sign-in, the payload executes — meaning it's stored in the database and reflected back unencoded wherever the full name is displayed (e.g., dashboard, welcome message, profile header).
This is stored XSS, not reflected — the payload persists and fires on every page load where the name is rendered, for the account owner and potentially any other user/admin who views that profile.

## Steps to Reproduce

1. Go to http://kzlabs.in/60.php and register a new account.
2. In the "Full Name" field, enter: '">shivay<ImG SrC=X OnErrOr=confirm(1)>
3. Complete signup and log in.
4. On the post-login page, a confirm() dialog pops with 1, confirming the payload executed from stored data.
5. Check page source on the dashboard/welcome page — the full name field is rendered without encoding, letting the <img onerror> tag execute.

## Proof of Concept

<img width="1915" height="682" alt="image" src="https://github.com/user-attachments/assets/3f65c9f7-c82f-417f-98ae-b8cb3972342e" />



## Impact

This is more serious than the reflected findings since it's persistent — the payload fires every time the page renders the name, no crafted link needed. If an admin panel, support dashboard, or any other user view displays this full name field (e.g., in a user list, chat, or order details), the payload would also fire in their session — leading to cookie theft, session hijacking, or admin account takeover depending on who views it.

## Remediation

1. HTML-encode the full name value everywhere it's rendered (", ', <, >), not just at input time.
2. Validate/restrict allowed characters in the name field server-side (letters, spaces, basic punctuation only — no <, >, " etc.).
3. Add a CSP to restrict inline event handler execution as a backup layer.
4. Set HttpOnly + Secure on session cookies to limit damage if this fires in another user's session.

