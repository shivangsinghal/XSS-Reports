## Title
Reflected XSS in search parameter — 55.php

## Affected URL
https://kzlabs.in/55.php?search=%22-alert(1)-%22

## Description

The search parameter gets reflected back into a JS string on the page without being escaped. Payload "-alert(1)-" closes the string early and lets alert(1) run as actual JS. No filtering on quotes.

## Steps to Reproduce

1. Visit: https://kzlabs.in/55.php?search=%22-alert(1)-%22
2. Alert box pops with 1, confirming execution.
3. View page source — you'll see the input landing inside a JS string literal, and the injected quote breaks out of it.

## Impact

Attacker can run arbitrary JS in a victim's browser by sending them this link. Cookies, session tokens, page content all fair game if session cookies aren't HttpOnly. Could be used for account takeover, phishing overlays, or redirecting users elsewhere.

## Remediation

1. Escape user input properly before putting it into a JS context (not just HTML-encode — quotes/backslashes need JS-specific escaping).
2. Better don't inject user input into inline <script> at all. Pass it via a data attribute and read it with JS instead.
3. Add a CSP to limit inline script execution as a backup layer.
4. Set HttpOnly on cookies so even if this fires, session hijacking isn't trivial.
