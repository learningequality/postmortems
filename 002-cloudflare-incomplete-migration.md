## Owner: @aronasorman
## Involved:
- @jamalex
## Meeting scheduled for: Waived by @aronasorman
## Call recording: 

# Overview

In the morning of Sept. 18th, 2017, A couple of our sites went down for a few
hours due to a migration to Cloudflare that's incompatible with our gatehouse
system. It affected every user using at least learningequality.org, pantry, community and
kolibri demo servers.

# What happened

During the week fo Sept. 18th, we were preparing for a larger than normal influx
of users due to being showcased by Riot Games (the creators of League of
Legends). To prepare for this, we did a quick integration of Cloudflare, a free
CDN and DDoS protection service, with learningequality.org.

For Cloudflare to fully do its job, we have to switch to using its nameservers,
and have it act as a proxy to our servers. That makes sense, since for DDoS
protection, we want our external users to see Cloudflare's IP, and not ours.

Cloudflare has an auto-detect domains feature that will try to see what domains
you already have. @aronasorman looked over it without being thorough, and
trusted that it would detect all subdomains we have. The main thing it missed
was a wildcard we had on namecheap (our registrar), that had a wildcard on any
learningequality.org subdomain that did not match any of the explicitly
registered domains.

Upon nameserver switch, we started getting loops on studio.learningequality.org,
where it would sometimes work (i.e. go through cloudflare and respond properly,
or go through gatehouse and redirect to learningequality.org because gatehouse
does not handle studio.learningequality.org).

In addition to that, we started having issues with other sites once DNS has propagated.
Pantry, community and others were unavailable to users and were returning domain not found.

# Root causes

There are two issues at play:

1. The DNS loop of the studio domain sometimes using gatehouse, and sometimes
using the cloudflare proxy (which works).
1. The unavailability of certain LE sites.

For the DNS loop issue, the problem was that namecheap implemented a wildcard
for all subdomains not explicitly listed. This led to certain DNS servers (like
google's) to sometimes serve the explicitly defined A record for a subdomain
(e.g. studio.learningequality.org pointing to specific IP) or it pointed to our
catch-all wildcard to gatehouse, which just redirected to learningequality.org.

Turning off the wildcard, however, led to the second issue -- since some sites
were not explicitly declared in our list of subdomains (only implicitly, through
the wildcard), that made Cloudflare miss those implicitly declared subdomains.

# Resolution

We disabled the Cloudflare migration first and reverted back to Namecheap's name service.

After that, we did a more careful migration, going through every subdomain we
had, and adding them to Cloudflare's nameserver. We then also went through every
domain that gatehouse is meant to handle (through ansible-playbooks), and added
them as well to Cloudflare (still pointing to gatehouse). Once we're sure that all
domains are handled, we initiated the switch to Cloudflare's nameservers again. Success!

# Corrective actions

- Avoid using toplevel wildcards on the DNS level.
- Check every subdomain we have and make sure we've migrated every subdomain, before switching nameservers.
- Use an uptime service to monitor *ALL* of our domains, to check for uptime.

# Notes to the owner

After running the meeting and finishing the postmortem, make sure that the following are done:

- [x] Push this document to github.com/learningequality/postmortems
- [] Schedule any follow up meetings to fulfill the long term corrective actions
- [] File issues for all the corrective actions listed on the relevant repos
