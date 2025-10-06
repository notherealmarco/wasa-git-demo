# Practical Git – Exercise (WASA class 02/10/2025)

This lab walks you through the exact steps to reproduce the current state of this repository from scratch.
This is a practical exercise to get you familiar with the most common Git commands and workflows.

Tip: to make the history easier to read, define a handy alias:

```bash
git config --global alias.lg "log --graph --abbrev-commit --decorate --date=relative --format=format:'%C(bold blue)%h%C(reset) - %C(bold green)(%ar)%C(reset) %C(white)%s%C(reset) %C(dim white)- %an%C(reset)%C(bold yellow)%d%C(reset)' --all"
```

This alias allows you to run `git lg` to see a nice graphical log of your commits.

## 0) Configure your identity and start a new repo

In order to make commits, you need to tell Git your name and your e-mail address.
This can be done globally (once per machine) or per-repo (once per repo).

To set it globally, run:
```bash
git config --global user.name "Your awesome full name"
git config --global user.email "your@email.com"
```
The `--global` flag means that this applies to all your repos on your machine.

Now we are ready to create our first repository!
```bash
# Create a new directory, which will hold your repo
mkdir -p wasa-git-demo
cd wasa-git-demo

# Initialize the repository (this will tell Git that this folder is a repo)
git init

# By default, your initial branch is named 'master'. In some cases you may want to rename it to 'main' (e.g. if you use GitHub)
git branch -m main
```

## 1) Our first commits on main

We are going to create three files, each containing a translation of "Hello world!" in a different language, and commit them one by one.

Before making a commit, you need to stage the changes you want to include in the commit using `git add <file>`. Otherwise, the commit will not include those changes.
You can also track multiple files at once using `git add .` to stage all changes in the current directory and its subdirectories.

```bash
# Create the first file
# the following command will create a file named english.txt with the content "Hello world!". You can also create it with a text editor like VSCode.
echo "Hello world!" > english.txt
git add english.txt
git commit -m "Add a simple text file"

# Add Italian
echo "Ciao mondo!" > italian.txt
git add italian.txt
git commit -m "Add Italian translation"

# Add German
echo "Hallo Welt" > german.txt
git add german.txt
git commit -m "Add German translation"

# Inspect history
git lg
```

At this point you have three files and three commits on the `main` branch.

## 2) Out first branches

```bash
# Create a branch to try something
git branch experiment

# Switch to the newly created branch
git checkout experiment

# Add Spanish on experiment
echo "Hola mundo" > spanish.txt
git add spanish.txt
git commit -m "Add Spanish translation"

# Merge experiment into main (fast-forward)
git checkout main
git merge experiment
```

Now the `main` branch contains the Spanish translation, which was added on the `experiment` branch, and then merged into `main`.

This is very useful for trying out new ideas without affecting the main line of development. You can develop new features of your application using a separate branch, and only merge them into `main` when they are ready.

In this case, git will use the `fast-forward` strategy, because `main` has not changed since `experiment` was created. This means that `main` will simply be moved forward to point to the same commit as `experiment`. No new commit is created.

Let's now try to do a merge using a different (non `fast-forward`) strategy.

```bash
# From main, create another branch that reverts Spanish
git branch dev1

# Switch to the newly created branch
git checkout dev1

# Remove spanish.txt
rm spanish.txt

# Track the change and commit
git add spanish.txt
git commit -m "Remove Spanish translation"

# Switch back to main and merge dev1 (explicitly telling git to not use fast-forward)
git switch main

git merge --no-ff dev1
```

A text editor may open to ask you to edit the default merge commit message. You can edit it or just save and close the editor to keep the default message.

When merging with a strategy different from `fast-forward`, git creates a new commit that has two parents: the current commit on `main` and the latest commit on `dev1`. We can see this in the history by running `git lg`.


## 3) Handling merge conflicts

We’ll create two branches that both change `english.txt` in different ways.

```bash
# Branch dev4 from main and change english.txt one way
git branch dev3
git checkout dev3

# Now let's modify english.txt, from "Hello world!" to "Hello"
echo "Hello" > english.txt

# Stage and commit the change
git add english.txt
git commit -m "Change English translation"

# Go back to main
# english.txt will go back to "Hello world!"
git checkout main

# Create a new branch called dev4 (starting again from main) for another change
git branch dev4
git checkout dev4

# Modify english.txt, from "Hello world!" to "world!"
echo "world!" > english.txt

# Stage and commit the change
git add english.txt
git commit -m "Update English translation"
```

Now we have two branches, `dev3` and `dev4`, that both changed `english.txt` in different ways.
We are currently on `dev4`. Let's switch back to `main` and merge `dev3` first.

```bash
# Switch back to main
git checkout main

# Merge dev3 into main (fast-forward will be used)
git merge --ff-only dev3

# Look at the history
git lg
```

`main` now contains the `dev3` version (“Hello World”). The `dev4` branch still contains its own conflicting version (“Hello world!!”).

### 3.1) Creating the conflict

```bash
# Merge dev4 into main; this should conflict on english.txt
git merge dev4
```

Now you are in a merge conflict state. Git was not able to automatically merge the changes from `dev4` into `main` because both branches modified the same line in `english.txt` in different ways, and it needs your help to resolve the conflict.

You will need to open english.txt and resolve the conflict to the final text.
In the file you will se something like this:

```
<<<<<<< HEAD
Hello
=======
world!
>>>>>>> dev4
```

You should edit the file to contain only the final chosen line. In our case, we want it to be again "Hello world!".

```bash
# Final chosen line: Hello world!
echo "Hello world!" > english.txt

# Stage the resolved file and commit the merge
git add english.txt
git commit -m "Merge dev4 into main"

# See the resulting history graph
git lg
```

You’ve resolved the conflict by keeping a clean final line: `Hello world!`.

## 4) Git remotes: push and fetch

Git is more powerful when you can share your work with others, or back it up on a remote server. This is done using remotes.
There are plenty of them, with the most popular being GitHub, GitLab, BitBucket, Gitea and Codeberg.
You could even host your own remote server if you wish.

In this example, we are going to use GitHub.
To create a new empty repository on GitHub, go to `https://github.com/new` and provide a name. You can choose to make it public (visible to everyone) or private (visible only to you and people you explicitly share it with).

For a public repository, you may want to consider adding a license, this will allow others to use your code under the terms you choose.
Without adding a license, your code is technically "all rights reserved" and others cannot legally use it.
I recommend to check out [choosealicense.com](https://choosealicense.com/) to help you decide which license is best for your project.

You can add a license directly on GitHub when creating the repository, or you can add it later by creating a `LICENSE` file in your repository, with the content of the license you choose.

Once the repository is created, you are ready to link your local repository to the remote one. You can do it in two ways: HTTPS or SSH. I really recommend using SSH, as it is simpler and it uses key-based authentication, so you don't need to configure a credential manager or enter your username and password every time you push or fetch.

### 4.1) Setting up SSH keys (if you haven't done it yet)
If you don't have an SSH key pair yet, you can create one by running `ssh-keygen -t ed25519`. ed25519 is a modern and secure algorithm for generating SSH keys. If your system does not support ed25519, you can use `rsa` instead: `ssh-keygen -t rsa -b 4096`.

When prompted, leave the file path as default (just press Enter). You can then choose to set a passphrase for your key, which adds an extra layer of security. If you set a passphrase, you will need to enter it every time you use the key, unless you use an SSH agent.
If you don't want to set a passphrase, just press Enter when prompted, and confirm by pressing Enter again.
It is safe enough to leave the passphrase empty for most use cases, especially if you are the only one using your machine.

> [!IMPORTANT]  
> You can use the same SSH key for multiple services (e.g. GitHub, GitLab BitBucket, etc.). You don't need to create a new key for each service. The best practice is to use a single key per machine, unless you have a specific reason to use multiple keys (e.g. different keys for work and personal projects).

Finally, a new key pair will be created in the `~/.ssh` directory (or `C:\Users\<YourUsername>\.ssh` on Windows). The private key is stored in a file named `id_ed25519` (or `id_rsa` for RSA keys), and the public key is stored in a file named `id_ed25519.pub` (or `id_rsa.pub` for RSA keys).

Now you need to add the public key to your GitHub account. You can do this by copying the content of the public key file and adding it to your GitHub account settings at [https://github.com/settings/keys](https://github.com/settings/keys).

> [!CAUTION]
> NEVER send your private key to anyone or upload it anywhere. The private key is used to prove your identity, and if someone else gets access to it, they can impersonate you and hack into your GitHub account. GitHub and other services will only need your public key to verify your identity.

### 4.2) Adding the remote and pushing your changes to GitHub

From your GitHub repository page, copy the SSH URL (it should look something like `git@github.com:notherealmarco/wasa-git-demo.git`). Now you can add the remote to your local repository:

```bash
git remote add origin <your-repo-ssh-url>
```

Here, `origin` is the name of the remote. You can choose any name you want, but origin is the convention for the main remote repository.
In complex projects, you may have multiple remotes (e.g. one for the main repository, one for a fork, etc.).

Finally, you are ready to push your changes to the remote repository.

# Push the branch main to the remote named origin
git push origin main

# Push every local branch (this includes main too)
git push --all origin
```

To avoid specifying the remote and branch every time you push, you can set the upstream branch for your local branches using the `--set-upstream` (or the short alias `-u`) flag the first time you push each branch:

```bash
git push --set-upstream origin <branch-name>
```

From now on, you can simply run `git push` to push the current branch to the remote named `origin`.

If you open your GitHub repository page, you should see all your commits and branches there.

### 4.3) Pulling remote changes

When you are working in groups (or if you are working on multiple machines), you will need to pull remote changes to keep your local repository up to date.

The `git pull` command is used to fetch and integrate changes from a remote repository into your current branch.
It is actually combination of two commands: `git fetch` followed by `git merge`.

> [!NOTE]
> `git pull` will only update the branch you are currently on.

## 5) Writing meaningfull commit messages

Writing clear and consistent commit messages is essential for understanding the history of a project, collaborating with others, and automating tasks like changelog generation or semantic versioning. A good commit message should explain what changed and why, not just how.

A common structure is `<type>(<scope>): <short summary>`, where:
* **type:** Describes the kind of change, e.g. feat (new feature), fix (bug fix), docs (documentation), refactor (code improvement without changing behavior), chore (maintenance), etc.
* **scope:** Optional, specifies the part of the code affected, e.g. backend, ui, database...
* **short summary:** A concise description (ideally <50 characters) of the change.

Examples:
```
# Feature addition
git commit -m "feat(translation): add Spanish greeting"

# Bug fix
git commit -m "fix(english.txt): correct typo in greeting"

# Documentation update
git commit -m "docs: add instructions for GitHub SSH setup"

# Refactor
git commit -m "refactor: simplify commit alias for log view"
```

### 5.1) Best practices

* Keep the summary **short and imperative**: "Add", "Fix", "Update", not "Added" or "Fixed".
* Use the body to **explain why**, not what — the code itself shows what changed.
* **Group related changes** into a single commit rather than spreading them across multiple commits.
* Avoid vague messages like "update" or "fix".

#### Recommended reads
* [The commit convention](https://www.conventionalcommits.org/en/v1.0.0/#summary)
* [How to Write Better Git Commit Messages – A Step-By-Step Guide](https://www.freecodecamp.org/news/how-to-write-better-git-commit-messages/)


---
Congratulations! You have completed your first practical Git exercise.
This exercise covered the basics of Git, and includes everything you need to know to work on your WASA project.

Of course, there is much more to learn about Git, but this should be enough to get you started. If you want to learn more, I recommend reading the [Pro Git book](https://git-scm.com/book/en/v2) (which is free and available online) or the [Git documentation](https://git-scm.com/doc).

Something I really recommend you to learn, if you are working on a group project, is `git rebase`. Rebasing is a powerful tool that allows you to keep a clean and linear history, which is very useful when working in groups. It differs from merging, as it rewrites the commit history instead of creating a new merge commit. In big group projects, this can be very useful to avoid unnecessary merge commits and keep the history easier to read.
However, it is a bit more advanced, so I recommend to learn it after you are comfortable with the basics of Git.
