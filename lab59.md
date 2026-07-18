## Title

Reflected XSS in URL path parameter (comment ID) — 59.php

## Affected URL

http://kzlabs.in/59.php/svc/shreddit/api/comments/askreddit/t3_u9po1l'"><ImG SrC=X OnErrOr=confirm(1)>/t1_i5sxroa

## Description

A path segment in the comments API route (t3_u9po1l...) is reflected back into the page without encoding. Payload '"><ImG SrC=X OnErrOr=confirm(1)> closes out the current attribute/tag context and injects an <img> tag with an onerror handler that fires on load. This looks like a Reddit-style comment/post ID being pulled straight from the URL path and rendered somewhere on the page (likely in a breadcrumb, canonical link, or "viewing thread X" element) without sanitization.

## Steps to Reproduce

1. Visit: http://kzlabs.in/59.php/svc/shreddit/api/comments/askreddit/t3_u9po1l'"><ImG SrC=X OnErrOr=confirm(1)>/t1_i5sxroa
2. A confirm() dialog pops with 1, confirming execution.
3. View source — the quote/bracket sequence breaks out of the original attribute where the thread/comment ID is reflected, and the <img onerror> tag runs directly after it.

## Proof of Concept

<img width="1913" height="535" alt="image" src="https://github.com/user-attachments/assets/8b189366-69be-4f3a-96f2-7878e97a38db" />


## Impact
Anyone who clicks a crafted thread/comment link like this gets arbitrary JS executed in their browser under the kzlabs.in origin. This could be used for cookie theft, session hijacking, or redirecting/phishing users browsing shared comment links — especially risky if this route is publicly shared/indexed like real Reddit permalinks tend to be.

## Remediation

1. HTML-encode the path segment value before reflecting it anywhere in the page (", ', <, >).
2. Validate the ID format at the routing layer (e.g., t3_[a-z0-9]+ only) and reject anything that doesn't match before processing further.
3. Add a CSP to restrict inline event handler execution as a backup layer.
4. Set HttpOnly + Secure on session cookies.
