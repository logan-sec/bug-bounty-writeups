## Summary

I identified a behavior where a user account could be temporarily locked for approximately 15 minutes using only an email address, without needing valid credentials or prior authentication to victims account.
While the impact was limited in duration, this represented a user-targeted denial-of-service condition triggered through an unexpected endpoint.

## Vulnerability Class: 

  - Business Logic / Authentication Flow Abuse

  - Account Lockout (Unauthenticated Trigger)

## Affected Area:

Change email feature, in example.com account settings page.

No domain names, endpoints, or identifiers are included in here. I will refer to the target application as "example.com"

## Discovery Process

After completing full manual reconnaissance of the application (hours of navigating the UI and observing behavior), I tested the application **feature-by-feature** rather than vulnerability-by-vulnerability. In my experience, account settings often contain business logic issues, so I started there.

With Burp Intercept enabled, I captured normal account update traffic. Initial actions (such as updating profile details) resulted in expected GraphQL account update requests.

When reviewing the **change email** flow, the UI required:
- Current email (read-only)
- New email
- Account password

However, inspection of the corresponding request revealed that the **current email value was still modifiable** at the request level.

I replayed the request with:
- A random email → failed  
- Another account’s email → failed  
- Another account’s email **and password** → request accepted

Although the change only affected my own account (not the victim’s), this behavior suggested that the endpoint was validating passwords **without authenticating the associated account context**.

This raised concern about rate-limiting and lockout behavior. To test this, I treated the second account as a victim and attempted password guessing against the email-change endpoint using Burp Intruder.

Even when the correct password was supplied, responses continued to return errors. At that point, I logged into the second account and observed that it had entered a **temporary locked state**.

This confirmed that repeated unauthenticated interactions with the email-based flow could trigger an **account lockout**, requiring only knowledge of the victim’s email address. The lockout automatically resolved after approximately 15 minutes.

## Impact:

  - Temporary denial of access for legitimate users

  - Potential abuse for targeted harassment

  - No account takeover observed

  - Lockout duration was time-based and automatically resolved

While not a high-severity issue, the behavior could be abused at scale or used to disrupt specific users.

## Mitigation Suggestions:

  - Require proof of email ownership before triggering lockout states
  - Do not allow actions to be performed on another account with correct email and password if the session does not match.

## Disclosure Outcome:

The report was reviewed and ultimately marked N/A.
Despite this, the finding helped reinforce the importance of business logic testing beyond classic vulnerability classes, particularly around authentication and state changing flows.

## Lessons Learned:

  - Account lockouts are not always tied to passwords or login flows

  - Business logic issues can exist even when technical controls are present

  - Clearly documenting impact and abuse scenarios is critical, even for low-severity findings
