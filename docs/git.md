# Git

A guide on using Git, a version control system.

## Changing Author Data In All Commits

So you've changed your name, email, etc. and would like to practice good opsec by replacing all of your old commit author data. Good for you!
With regular `git` commands, it is difficult to do this in bulk and still maintain things like commit date. Enter `git-filter-repo`.
Install `git-filter-repo` with your distribution's package manager and then make the following files in the directory above your repo (not inside your repo):

- `mailmap.txt`
- `name_change.txt`

In `mailmap.txt`, put the following:

```text
New Name <new@email.com>
New Name <new@email.com> Old Name <old@email.com>
<new@email.com> <old@email.com>
```

In `name_change.txt`, put the following:

```text
old@email.com==>new@email.com
Old Name==>New Name
old_username==>new_username
```

You can make as many of these entries as you'd like in these files in the event you're dealing with multiple old names/emails.

Finally, open your CLI in the repository you'd like to make changes to and run the following:

```shell
git filter-repo --mailmap ../mailmap.txt && \
git filter-repo --replace-text ../name_change.txt && \
git filter-repo --replace-message ../name_change.txt
```

Note that for safety reasons, `git-filter-repo` removes your `origin` remote so you will need to add it back with:

```shell
git remote add origin <git_repo>
```

Review your commits, make sure everything looks correct. Then, you can push with:

```shell
git push origin <branch> -f
```

That should be it! For GitHub specifically, you will probably need to allow force-pushing in your branch settings before the above command will work.
