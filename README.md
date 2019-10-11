This repo houses tooling, docs, and an archive of old Gerrit CRs for Joyent
Engineering's migration *away* from the Gerrit at https://cr.joyent.us to using
GitHub PRs.

See [MANTA-4594](https://jira.joyent.us/browse/MANTA-4594) ([bugview
link](https://smartos.org/bugview/MANTA-4594)) for full details.

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

4.  Update the repo's README.md and/or CONTRIBUTING.md to remove any language
    about Gerrit usage, if the repo has it.

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

6.  Modify the jenkins job for this repository to remove gerrit references.
    If you do not have permissions to do this, ping us in ~mib.

    - Visit `https://jenkins.joyent.us/job/<your job>/configure`
    - remove the `$BRANCH` parameter description text that describes
      how to build from gerrit
    - In the 'Source Code Management' section, click the red 'X' to remove
      the `https://cr.joyent.us/joyent/<repository>` box

    If you do not do this, then Jenkins build of this project will fail,
    with a message similar to:

    ```
        Fetching upstream changes from https://cr.joyent.us/joyent/triton-cmon-agent.git
    > git --version # timeout=10
    > git fetch --tags --progress https://cr.joyent.us/joyent/triton-cmon-agent.git +refs/changes/*:refs/remotes/cr/*
    ERROR: Error fetching remote repo 'cr'
    hudson.plugins.git.GitException: Failed to fetch from https://cr.joyent.us/joyent/triton-cmon-agent.git
        at hudson.plugins.git.GitSCM.fetchFrom(GitSCM.java:894)
        at hudson.plugins.git.GitSCM.retrieveChanges(GitSCM.java:1161)
        at hudson.plugins.git.GitSCM.checkout(GitSCM.java:1192)
        at hudson.scm.SCM.checkout(SCM.java:504)
        at hudson.model.AbstractProject.checkout(AbstractProject.java:1208)
        at hudson.model.AbstractBuild$AbstractBuildExecution.defaultCheckout(AbstractBuild.java:574)
        at jenkins.scm.SCMCheckoutStrategy.checkout(SCMCheckoutStrategy.java:86)
        at hudson.model.AbstractBuild$AbstractBuildExecution.run(AbstractBuild.java:499)
        at hudson.model.Run.execute(Run.java:1815)
        at hudson.model.FreeStyleBuild.run(FreeStyleBuild.java:43)
        at hudson.model.ResourceController.execute(ResourceController.java:97)
        at hudson.model.Executor.run(Executor.java:429)
    Caused by: hudson.plugins.git.GitException: Command "git fetch --tags --progress https://cr.joyent.us/joyent/triton-cmon-agent.git +refs/changes/*:refs/remotes/cr/*" returned status code 128:
    stdout:
    stderr: fatal: remote error: Git repository not found

        at org.jenkinsci.plugins.gitclient.CliGitAPIImpl.launchCommandIn(CliGitAPIImpl.java:2160)
        at org.jenkinsci.plugins.gitclient.CliGitAPIImpl.launchCommandWithCredentials(CliGitAPIImpl.java:1852)
        at org.jenkinsci.plugins.gitclient.CliGitAPIImpl.access$500(CliGitAPIImpl.java:78)
        .
        .
        .
    ```

7.  Add a comment to [TOOLS-2327](https://jira.joyent.us/browse/TOOLS-2327)
    stating that you've moved the repo. Please mention the unmoved CRs if
    any remain for your repo.


## How to move an archived CR to a PR

All cr.joyent.us CRs are periodically archived (by Trent, manually) at
https://github.com/joyent/gerrit-migration/tree/master/archive
This repo provides a tool, `cr2pr`, to migrate an open CR from the archive
to a GitHub PR on the appropriate "joyent" org repo.

Here is a query of PRs that have been migrated using this tool:
<https://github.com/pulls?q=is%3Apr+user%3Ajoyent+migrated-from-gerrit>

WARNING: If your CR is new, or recently updated, the archive might not have
your CR (in which case `cr2pr` will fail) or the archive might have an outdated
patch set (in which case `cr2pr` will **use the outdated patch set**).
Please either check with Trent or confirm that the "./archive/$CRNUM/" dir
has the expected number of "patchSet-N.diff" files.

To move a CR to a PR:

0. Assumptions/Prerequisites:

    - You have `json` on your PATH (`npm install -g json`)
    - You have the `hub` tool on your PATH
      (<https://github.com/github/hub#installation>).
    - You have push access to the relevant "joyent/NAME" repo.
    - `ssh cr gerrit --help` works for you.

1. Use one of the following to run `cr2pr`:

    - download and run it:

            curl -ksSL -O https://raw.githubusercontent.com/joyent/gerrit-migration/master/bin/cr2pr
            chmod +x ./cr2pr
            ./cr2pr CR-NUM

    - or, run it from a clone of this repo (warning: this repo is big):

            git clone git@github.com:joyent/gerrit-migration.git
            ./bin/cr2pr CR-NUM


See https://gist.github.com/trentm/6188f14cd7487ac288c85080bfa7c3f3 for what
an example run looks like.


## How to update the cr.joyent.us archive

Trent is the only one that should need to do this.

0. Optional. If you are making huge updates to the archives (e.g. starting from
   an empty archive), then you'll want to prefetch all the repos from
   cr.joyent.us via:

    ```
    ssh cr gerrit ls-projects | sed 1d > archive/remaining-projects.txt
    export CACHE_DIR=/var/tmp/cr.joyent.us-archive/repos
    cat archive/remaining-projects.txt | while read p; do
        if [[ "$p" == "joyent/postgres" ]]; then
            echo "";
            echo "# skip $p (broken clone from cr.joyent.us)"
        elif [[ ! -d $CACHE_DIR/$p.git ]]; then
            echo "";
            echo "# clone $p";
            dir=$CACHE_DIR/$(dirname $p);
            mkdir -p $dir;
            (cd $dir && git clone --mirror trentm@cr.joyent.us:$p.git);
        fi;
    done
    ```

1. Get a clone of this repo:

    ```
    git clone git@github.com:joyent/gerrit-migration.git
    cd gerrit-migration
    ```

2. Run the archive script:

    ```
    ./bin/cr.joyent.us-archive ./archive
    ```

    However cr.joyent.us is flaky. It will (a) times out or crashes in SSH API
    requests and (b) hang on "git clone" and "git fetch" frequently. I've
    settled on running the following and watching for hangs. When it
    hangs, hit `Ctrl+C` to abort and have it retry:

    ```
    # First edit the file and set this in the "globals/config" section:
    #      regenerateAllChanges = False
    vi bin/cr.joyent.us-archive

    # Then run this:
    while true; do
        echo ""; echo ""; echo "";
        ./bin/cr.joyent.us-archive ./archive;
        echo ""; echo "";
        echo "# will retry in 5 seconds"; sleep 5;
    done

    # Then remember to undo your script change.
    vi bin/cr.joyent.us-archive
    ```

3. Update list of remaining repos/projects and open CRs:

    ```
    ssh cr gerrit ls-projects | sed 1d > archive/remaining-projects.txt
    json -gaf archive/all-changes.json \
        -c this.open \
        -e 'this.subj=this.subject.split(" ")[0]' url project subj -o json-0 \
        | tabula -s project > archive/open-crs.txt
    ```

4. Git add and commit the updates:

    ```
    git add ./archive
    git commit archive -m "MANTA-4595: update cr.joyent.us archive to latest"
    git push origin master
    ```



## Statistics

Getting a list of remaining projects that have *no open CRs*:

    cat archive/open-crs.txt  | awk '{print $2}' | uniq > projects-with-open-crs.txt
    comm -23 archive/remaining-projects.txt projects-with-open-crs.txt > remaining-projects-without-crs.txt

Remaining projects *with open CRs*:

    comm -12 archive/remaining-projects.txt projects-with-open-crs.txt
