# Sushibar expired certificate postmortem

## Date
Start: Wednesday, 2018-08-22
End: Thursday, 2018-08-23

## Authors
Aron Asor

## Status
Resolved

## Summary
Sushi bars TLS certificate expired. This caused every sushi chef to error out, and people visiting the interface were blocked. We fixed this by piggybacking to Cloudflare’s wildcard certificate.

## Impact
Every sushi chef out there didn’t run. Everyone visiting the site couldn’t open it. 

## Root Causes
- The certificate expired 
- The certificate wasn’t auto renewing
- No notifications were triggered
- Certificate auto renewal wasn’t set up properly
- Renewal emails aren’t sent to a global email
- No one in the team knows how to renew the certificate

## Trigger
The deadline passed for certificate renewal

## Resolution
Since we don’t have access to sushi bar’s certificate, the workaround was to proxy the site through cloudflare. Since sushi bar forced https, we had to activate flexible SSL as well. That made cloudflare request the SSL cert on sushi bar, but not validate it. 

## Detection
Detection was through people telling us it wasn’t working. 

Action Items
- Set up alerting to warn us of expiring SSL

- sushibar’s instance should be hosted on the sushibar project, or the studio project at least

- Content pipeline team should be able to brainstorm and execute solutions to SSL problems

## Lessons Learned
- Know all your certificates, and when it’s expiring!
- Know how to renew certificates. Make sure it’s documented.

### What went well
The solution was fast once Aron managed to sit down and fix it.

### What went wrong
No one else had easy access to cloudflare initially. No one else knew how to renew the certificate.

### Where we got lucky
Cloudflare had a feature that proxied for a site through HTTPS, but didn’t need to verify it. That allowed us to continue using the expired certificate used by the sushibar host itself.

## Timeline

2018-08-22, 03:26pm: first noticed that sushibar.learningequality.org was
giving an error.

2018-08-22, 04:12pm: first check for the real problem. It was noted that the
certificate from lets's encrypt expired.

2018-08-22, 04:24pm: First pass proxying through cloudflare. It went into a
redirect loop due to cloudflare requesting the non-https version.

2018-08-23, 10:08am: Cloudflare was turned on again, and this time Full SSL
proxying was enabled, allowing cloudflare to request the site through HTTPS.
Issue resolved.

## Supporting information
