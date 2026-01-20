## Summary

I identified a behavior where a user account could be temporarily locked for approximately 15 minutes using only an email address, without needing valid credentials or prior authentication to victims account.
While the impact was limited in duration, this represented a user-targeted denial-of-service condition triggered through an unexpected endpoint.

## Vulnerability Class: 

  - Business Logic / Authentication Flow Abuse

  - Account Lockout (Unauthenticated Trigger)

## Affected Area:

Change email feature, in example.com account settings page.

No domain names, endpoints, or identifiers are included in here. I will refer to the target application as "example.com"

## Discovery Process:

I had just finished doing my manual recon for the entire application(hours and hours of clicking around example.com and note taking). Instead of testing for one type of vulnerability at a time; I test one feature at a time. 
So, I decided to start at account settings; oftentimes(in my experience) there can be lots of business logic bugs to be found here. I start by turning on Burp Intercept and normally capturing traffic. First, I changed my name 
from Attacker Attacker to John Attcker. I go to Burp http history and observe the response. It's just a basic GraphQL update account stuff. But then I go over to the change email button. This is where I notice something interesting; In the UI it has 
user current email (can't be changed), new email(enter new email), password(enter password to change email). After changing the email I go to http history and observe the request; here I notice you can change the current email! 
So, I sent request again with random email, failed. Try again with email for another account, failed. Try again with email for another account AND PASSWORD for that other account, Success! However, don't get to excited because it changed the email 
for my current account. So, I started thinking, I could bypass rate-limiting through this endpoint to enumerate victim passwords. So, I used that second account again(this time treated as victim), I tried brute-forcing the password field by using Burp Intruder. 
But even after the right password was entered after request 25, Response was still 400. At this point I did not understand why it was not working and was about to give up; when I decided to login to that other account. That's when I noticed UI said: "account is locked" 
So, while trying to brute-force Victim password I actually locked Victim out! Only lasted about 15 minutes though. 

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
