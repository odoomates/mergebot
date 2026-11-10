# Mergebot

The mergebot of the Odoo Mates repositories of Odoo modules: it merges pull requests on a comment, bumps the
version of the modules they change, thanks their authors and forward-ports them to the higher version branches.
It lives here once, and each repository calls it.

## Use it in a repository

`.github/workflows/mergebot.yml` of the repository:

```yaml
name: Mergebot

on:
  issue_comment:
    types: [created]
  pull_request_target:
    types: [closed]

permissions: {}

jobs:
  mergebot:
    permissions:
      contents: write
      pull-requests: write
      issues: write
      checks: read
    uses: odoomates/mergebot/.github/workflows/mergebot.yml@v1
    secrets: inherit
```

Secrets of the repository:

- `MERGEBOT_TOKEN`: a token of an admin of the repository, to rebase, bump and open the forward-ports. Without it
  the bot only merges.
- `MERGEBOT_LINKED_REPOS` (optional): the other repositories whose pull requests may be linked to this one,
  `owner/repo` separated by spaces.
- `MERGEBOT_MIGRATION_CHECK` (optional): `owner/repo@ref` of a migration checker run on the forward-ports.

Inputs: `bot-ref`, the release of the scripts (`v1` by default, keep it the same as in `uses:`), and `roles`, the
roles allowed to command the bot (`admin maintain write` by default).

## Commands

On the first line of a comment of a pull request, by a maintainer:

- `/mergebot r+` merges it once CI is green, then raises the patch version of the modules it changes; a branch
  behind its target branch is rebased first, and CI runs again before the merge;
- `/mergebot r+ minor` (or `major`) does the same with a minor or major version;
- `/mergebot r+ nobump` does the same without raising the version;
- `/mergebot r+ skipci` merges it now, without waiting for CI: only for an admin;
- `/mergebot r+ fw+` merges it, then forward-ports it: one pull request of its commits to every higher version
  branch, a draft where they do not apply; `/mergebot r+ fw+ 19.0` (or `fw+ upto 19.0`) stops at that branch;
- `/mergebot rebase` rebases the branch on its target branch;
- `/mergebot squash` makes the commits one commit, with the message of the first one;
- `/mergebot r-` cancels a merge waiting for CI, or for a linked pull request.

A line `Linked: owner/repo#12` in the description links two pull requests: `/mergebot r+` on one waits for
`/mergebot r+` on the other, and the second merges both.

The bot never checks out nor runs the code of a pull request: it runs its own scripts, from this repository, and
calls the GitHub API.

## Releases

The repositories call a release tag, `v1`. A change is merged on `main`, tried on a test repository calling
`@main`, then the tag is moved to it.
