# Metadata

Owner: @aronasorman

Involved:
  
Meeting scheduled for: Waived by @aronasorman

Call recording: <Link to the Hangout or recording>

# Overview

Monday, Nov. 13, 2017

During the instantschools-v2.learningequality.org initial server setup, users
were getting a "username or password is incorrect" error when trying to log in,
despite just creating an account a few minutes ago, due to a missing migration.
The affected were mostly the implementations team (@vuzy).

# What happened

One of the tasks we have to do that Monday morning, is to set up a new Kolibri
server for one of our partners. Setting up a new Kolibri server is
straightforward, but this special Kolibri server had an extra module that's
external to the repository. This module then has to be built first as a separate
whl file and then installed during Kolibri's build process, producing a PEX
file. We then use this PEX file as the main deployment artifact for live servers.

Installing and running the PEX file in our demo VPS instance was a breeze, but
after it was passed off for QA, we discovered that logging in one wasn't working.

The message was "username or password is incorrect", but when we read the
response from the server, the real error is from the database layer:

```
OperationalError: no such table: kolibri_instant_schools_plugin_phonetousernamemapping
```

This means that a database migration was missing, or didn't run correctly.

# Root causes

The real error was migrations for the external Kolibri module weren't getting bundled into
the .whl file. Because developers used a "developer install" of that external module during
dev, they didn't encounter this error. But files to be bundled in .whl packages need to be specified
explicitly, and is the reason why this was missed during development.

# Resolution

@aronasorman manually fixed this by manually copying all the migrations into the
.pex extraction location for the external module, and then running `kolibri
manage migrate`. Kolibri/Django then saw those migrations (despite being placed after install time),
and then added the corresponding tables and columns.

# Corrective actions

The proper fix, is to modify the MANIFEST.in file (the file used by Python to
determine which files to include into the .whl package) and change a misdeclared
line. Python will silently fail when a line has a trailing slash in the path,
and won't include any files indicated by the line. The fix was to remove the trailing slash.

See the full diff
[here](https://github.com/fle-internal/kolibri-instant-schools-plugin/commit/7b414f421b4612392e3da234b12786045ca417e3).


# Notes to the owner

After running the meeting and finishing the postmortem, make sure that the following are done:

- [x] Push this document to github.com/learningequality/postmortems
- [x] File issues for all the corrective actions listed on the relevant repos
