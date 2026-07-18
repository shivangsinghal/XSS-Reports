## Title

Stored XSS in article "Body" field (HTML editor) — 61.php

## Affected URL

http://kzlabs.in/61.php

## Description

The article creation feature on this page includes a "Body" input field with an HTML/code toggle switch, presumably meant for basic formatting. This field doesn't sanitize input before storing and rendering it. Payload '"><ImG SrC=X OnErrOr=confirm(1)> was entered into the body field with the HTML switch enabled, saved, and on rendering the article, the confirm() dialog fires — meaning the raw HTML/script gets stored and reflected back unencoded wherever the article is displayed.
This is stored XSS — the payload persists in the database and executes for anyone who views the published article, not just the author.

## Steps to Reproduce

1. Create an account at http://kzlabs.in/61.php and go to the article creation section.
2. In the "Body" field, toggle the HTML/code switch (allows raw tags).
3. Enter: '"><ImG SrC=X OnErrOr=confirm(1)>
4. Save/publish the article.
5. View the article (own page or wherever it renders) — a confirm() dialog pops with 1, confirming stored execution.

## Proof of Concept

<img width="1908" height="758" alt="image" src="https://github.com/user-attachments/assets/8670396f-5326-485c-a684-7640bce96648" />


## Impact

This is higher severity than reflected findings since the payload is persistent and requires no crafted link — anyone who views the article (other users, admins, moderators reviewing content) gets the payload executed in their browser automatically. This could lead to session/cookie theft, account takeover, or admin panel compromise if a privileged user (e.g., content moderator) opens the article for review.

## Remediation

1. HTML-encode/sanitize the body content on output, even if raw HTML input is intentionally allowed — use an allow-list based HTML sanitizer (e.g., DOMPurify server-side equivalent, or a strict tag/attribute whitelist) instead of storing and rendering raw user HTML.
2. If the "HTML mode" toggle is meant to support formatting only (bold, links, etc.), restrict allowed tags/attributes explicitly — block <script>, on* event handlers, <img> with onerror, etc.
3. Add a CSP to restrict inline event handler execution as a backup layer.
4. Set HttpOnly + Secure on session cookies to limit damage if this fires in another (e.g., admin) session.
