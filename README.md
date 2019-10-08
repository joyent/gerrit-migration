This repo houses tooling, docs, and an archive of old Gerrit CRs for Joyent
Engineering's migration *away* from the Gerrit at https://cr.joyent.us to using
GitHub PRs.

See [MANTA-4594](https://jira.joyent.us/browse/MANTA-4594) ([bugview
link](https://smartos.org/bugview/MANTA-4594))for full details.


## How to move a repo from cr.joyent.us (Gerrit) to GitHub PRs

We will use "joyent/node-manta" as the example project/repo in these notes.
Substitute with your project/repo.

1.  Look at any currently open CRs for the project and take note of the CR
    numbers. Visit <https://cr.joyent.us/#/q/is:open+project:joyent/node-manta>
    and/or run:

        ssh cr gerrit query --format=JSON -- is:open project:joyent/node-manta \
            | json -ga project url owner.email subject

    For each open CR:

    - If it can be certainly be adandoned, then do so.
    - If it can be fast tracked and merged before proceeding, then do so.

    Remaining CRs can be moved to PRs (TODO: process/docs for this),
    following the "How to move an archived CR to a PR" process below.
    Moving to PRs can be deferred to later.

2.  Adjust "master" branch protection rules in the GitHub repo settings.

    Our typical Gerrit import process involved restricting commits to
    the GitHub "master" branch to the "joyent-automation" account that
    cr.joyent.us uses to push to GitHub. We want to remove that.

    - Visit https://github.com/joyent/node-manta/settings/branches
    - Click "Edit" on the branch protection rule for "master"
    - Set checkboxes as follows:

            [x] Require pull request reviews before merging

                Required appoving reviews: 1

                [x] Dismiss stale pull request approvals when new commits are pushed
                [ ] Require review from Code Owners
                [ ] Restrict who can dismiss pull request reviews

            [x] Require status checks to pass before merging

                [x] Require branches to be up to date before merging

            [ ] Require signed commits

            [x] Include administrators

            [ ] Restrict who can push to matching branches

        That last one, "Restrict who can push ...", is the important one to
        **uncheck**. The others are open for discussion, but this is a
        starting point.

    - Click "Save changes"

3.  Turn *off* the "merge commit" option in the "Merge button" settings:
    https://github.com/joyent/node-manta/settings#merge-button-settings

        [ ] Allow merge commits     <--- This is the important one.
        [x] Allow squash merging
        [x] Allow rebase merging

    Notes:
    - If you believe "merge commits" should be allowed, please let's have a
      discussion rather and come to an understanding.
    - If you want to disable "Allow rebase merging", I am fine with that
      personally. Feel free to ask.

4.  Do a commit directly to the GitHub master that updates the language about
    Gerrit/PR usage in the README.md and/or CONTRIBUTING.md (if the repo
    has it).

    For example:

    - Replace this:
        ```
        This repository is part of the Joyent Triton project. See the [contribution
        guidelines](https://github.com/joyent/triton/blob/master/CONTRIBUTING.md) --
        *Triton does not use GitHub PRs* -- and general documentation at the main
        [Triton project](https://github.com/joyent/triton) page.
        ```
        with this:
        ```
        This repository is part of the Joyent Triton project. See the [contribution
        guidelines](https://github.com/joyent/triton/blob/master/CONTRIBUTING.md)
        and general documentation at the main [Triton
        project](https://github.com/joyent/triton) page.
        ```

    Note that most Manta repos just refer to a CONTRIBUTING.md on the
    joyent/manta repo, so there might not be a need to remove any Gerrit
    references from your repo.

5.  **If all open CRs for this repo have been handled,** then "hide" the repo
    in gerrit.

    - Visit https://cr.joyent.us/#/admin/projects/joyent/node-manta
    - Change `State: Hidden`
    - Click "Save Changes"

6.  Add a comment to [TOOLS-2327](https://jira.joyent.us/browse/TOOLS-2327)
    stating that you've moved the repo. Please mention the unmoved CRs if
    any remain for your repo.


## How to move an archived CR to a PR

All cr.joyent.us CRs have been archived at
https://github.com/joyent/gerrit-migration/tree/master/archive
The following is a best-effort and manual process for re-submitting a
Gerrit CR as a GitHub PR.

1. Get a clone of the archive:

        git clone https://github.com/joyent/gerrit-migration.git

2.  XXX
