---
title: 'Lab 0: Setup and Tooling'
layout: page
released: 2026-01-06
due: 2026-01-15
index: 0
---
# Lab 0: Setup and Tooling

We assume throughout these instructions, and the class, that you have access to a terminal on your computer (for example, the Terminal program on Mac). If you do not have a computer, please contact the course staff, and we will provide support on the machines administered by Khoury. 

## Installing requirements
You need to install a few dependencies before you can get started. This step differs depending on whether you are using MacOS, Linux or Windows. Please follow the appropriate steps below.

### Mac or Linux
- On MacOS, you can open a terminal by running the Terminal program (from `/Applications/Utilities/`). Make sure that you have that working.
  Similarly, all Linux distributions come with at least one or more terminal emulator, but the precise steps to run it depends on the desktop environment you are using. Make sure you know how to start it.
- Install [Docker](https://docs.docker.com/desktop/).

You should now be ready to create the CS5600 Docker container.
### Windows
We will be using [Windows Subsystem for Linux](https://learn.microsoft.com/en-us/windows/wsl/install) in this class. The order of steps matters in this case, so please follow them carefully.

**If you previously installed Docker, but are unsure if you used WSL, please uninstall it before following the process below.**

1. Start Powershell, which is installed by default on Linux. The commands that follow should be run in Powershell. You will be using Powershell as your terminal through the rest of class.
2. Install (or update to) WSL 2:

```sh
> wsl --install
> wsl --set-default-version 2
```

If your Windows version is before Windows 10 version 2004, then you need to follow these [instructions])([https://learn.microsoft.com/en-us/windows/wsl/install-manual](https://learn.microsoft.com/en-us/windows/wsl/install-manual)) to manually install WSL 2.

3. Install [Docker](https://docs.docker.com/desktop/). When prompted to logout and login, please **reboot**. NB: The reboot is essential here, do not skip it.
4. After the reboot, start Powershell again. Install Linux by running

```sh
> wsl --install -d ubuntu
> wsl --set-default ubuntu
```

5. Start WSL: in Powershell run:

```sh
> wsl
$
```

6. Verify that you have git installed by running the following within WSL (this should display a help message):

```sh
 git
```

7. Make files group-writable by default:

```sh
echo "umask 002" > ~/.bashrc
```

Everything you do below (including the Git setup, etc.) and in the rest of class should be done within WSL, which you start by running `wsl` in Powershell.

> If you are having trouble with WSL, it may be because you had a previous Ubuntu instance installed. You can check that with wsl -l -v. There should be one entry (the one you installed above). If there is more than one, you should rename or back it up, and ensure that you are running the newly installed Ubuntu.

## Git and GitHub
We are going to use [GitHub](https://github.com/) and [`Git`](http://git-scm.com/) for distributing and collecting assignments. Please make sure you have `git` installed on your machine.

### What is git?
Git was developed by Linus Torvalds for development of the Linux kernel. It’s a distributed version control system, which means it supports many local repositories, which each track changes and can synchronize with each other in a peer-to-peer fashion. It’s the best widely-available version control system, and certainly the most widely used. For information on how to use Git, see:
- [Eddie Kohler’s excellent notes on Git](https://cs61.seas.harvard.edu/site/ref/git/).
- [GitHub Guides: Git Handbook](https://guides.github.com/introduction/git-handbook/)

For the workflow in GitHub:
- [GitHub Guides: Hello World](https://guides.github.com/activities/hello-world/)

### Getting set up on GitHub
You will fetch lab source files, and submit your code, by pulling from, and pushing to, [GitHub](https://github.com/). Although the design of Git is fundamentally peer-to-peer, it’s helpful to think of GitHub as a Git server that stores the authoritative or externally-visible versions of your Git repositories. GitHub is much more than that; modern developer teams use it (or its competitor GitLab) an essential tool for collaboration, review, project management, issue tracking, and more.

Many actions on GitHub will require authenticating yourself--- proving that you should have access to your repositories. The best way to do that involves an _SSH key_, a secret key stored on your computer that defines your identity to GitHub, and an _SSH agent_, a program that remembers the identity so that you don’t have to type your password all the time. Below are instructions:

1. If you don’t have a GitHub account, sign up for one [here](https://github.com/join). Probably you want a Free plan.
2. Create or configure an SSH key pair using [GitHub’s instructions](https://docs.github.com/en/authentication/connecting-to-github-with-ssh). To summarize:
    - Create the key (if you don’t have one already) using the `ssh-keygen` program:
```sh
ssh-keygen -t rsa -b 2048
```
        Then press enter to use the default file path and key name (should be `~/.ssh/id_rsa`), and choose a password. Your public key should now be in the file `~/.ssh/id_rsa.pub`.
        
    - Run `cat ~/.ssh/id_rsa.pub` to display your public key.
    - Copy your public key to the clipboard (select the text on the screen, and copy it to the clipboard; on a Mac, you can also use `pbcopy`, and on Windows you can use `clip`).
    - In GitHub, go to your profile (accessible via the upper-rightmost link; for accounts without an image that you set, this looks like a bunch of pixels). Then select Settings. Then on the left, select “SSH and GPG keys”. Then press “New SSH key”. Paste the contents of your `~/.ssh/id_rsa.pub` into the “Key” section. Give the key a sensible title, and press “Add SSH key”.

3. Use `ssh-agent` so that you don’t have to type your password every time you use the key. Typically the following is enough:
    
```sh
 ssh-add
```
    

but you may need to explicitly invoke `ssh-agent` and/or type `ssh-add ~/.ssh/<key_name>`, where `<key_name>` is the name of the key that you created above.

**About SSH identities.** An SSH identity is stored in two files, a public key with a name like `/Users/yourname/.ssh/id_rsa.pub` and a private key with a name like `/Users/yourname/.ssh/id_rsa`. The private key is kept secret – you should never upload the private key to a shared service – while the public key can be uploaded anywhere you like, including your GitHub account. The public and private keys are a matched pair. Services like GitHub verify your identity using a mathematical protocol: your computer essentially proves that it has access to the private key corresponding to your public key, after which GitHub “knows” that your computer speaks for you.

**If you use multiple computers to do your labs**: You’ll need to configure an identity on each of these computers. You can do this either by creating and configuring multiple SSH keys, following steps 2 and 3 above. That is the safer approach. An alternative is to copy the public and private keys between the computers. This is an easier approach, but if you don’t trust the computer you are copying to, you should not do this, because it leaves you vulnerable to being impersonated. In any case, if copying, you need to get the file permissions correct. Here is a way to do so:

```sh
$ mkdir -p ~/.ssh
... in here, copy public and private key files into ~/.ssh ...
$ chmod -R go-rwx ~/.ssh   # removes group and world permissions from everything
$ chmod go+r ~/.ssh/*.pub  # add back group and world readability for public keys
```

### Configure Git

You should also tell your `git` installation your name and email, if you haven’t already. This will ensure that you are recorded as the author of your code. For the `user.email` option, use your NYU email address:

```
git config --global user.name "FIRST_NAME LAST_NAME"
git config --global user.email "netid@northeastern.edu
```

## Getting the labs repository

Labs will be released using the [neu-sea-cs5600-labs](https://github.com/XXX) repository.

Please [click this link](https://classroom.github.com/XXX) to create **your own private clone** of the labs repository; this clone lives on (is hosted by) GitHub. Once that clone exists, you will perform a further clone to get that private repository onto your machine. You’ll do your work on your machine, and then push your work to the GitHub-hosted private repository to save our work, and to allow us to grade it.

Here’s how it should work:

1. Click the link above.
2. Log in to GitHub.
3. Provide a name.
4. Accept the assignment.
5. Refresh the page until the assignment repository is ready.

These steps should create a new repository for your. For instance, if your username name is `foomoo67`, you should now have a repository on GitHub called `neu-sea-cs5600/labs-spr26-002-foomoo67`.

**Note: GitHub Classroom may tell you to create a new repository on the command line, create a new branch, and make your first commit with a README file. _Do not do this._ It will introduce merge conflicts later on.**

### Creating a local clone
Here’s how to get a local clone of your private repo on your machine. Assuming your SSH identity is set up properly, follow the steps below.

- **Clone “your” lab repo**. Run the next command in a place on your computer where you fill find it. This will set up `cs5600` as THE directory where you do your labs work, so make sure it’s somewhere you can get to easily:
    
```sh
$ git clone git@github.com:neu-sea-cs5600/labs-spr26-002-<Your-GitHub-Username>.git cs5600
Cloning into ....
warning: You appear to have cloned an empty repository.
```
    
    Note that the `git@github.com:....` can be obtained on GitHub by visiting a link like `https://github.com/neu-sea-cs5600/labs-spr26-002-<Your-GitHub-Username>`, and then clicking the “Clone or download” button. You want to clone using SSH, not HTTPS, so you might need to click “Use SSH”.
    
    At this point, all you have is an empty repository, so you need to get the lab files. We’ll do that next.
    
    **Troubleshooting:** If you get “The connection timed out”, see the Git FAQ below.
    
- **Set up the upstream repo**: The lab skeleton code is kept in the repo `https://github.com/XXX/labs`, managed by the course staff. Therefore, the first thing you need to do is to set up your own lab repo to track the changes made in the `labs` repo. In the git world, `XXX/labs` would be the “upstream” repo from which changes should “flow” into your own lab repo.
    
    Type `git remote add` to add the upstream repo, and `git remote -v` to check that the right repo is indeed an upstream for your own lab repo.
    
```
$ cd cs5600
$ git remote add upstream https://github.com/XXX/labs.git
$ git remote -v
origin git@github.com:XXX/labs-spr26-002-<YourGithubUsername>.git (fetch)
origin git@github.com:XXX/labs-spr26-002-<YourGithubUsername>.git (push)
upstream https://github.com/XXX/labs.git (fetch)
upstream https://github.com/XXX/labs.git (push)
```
    
    Now fetch the commits from upstream:
    
```
$ git fetch upstream
remote: Enumerating objects: 13, done.
.....
From https://github.com/XXX/labs
* [new branch]      main       -> upstream/main
```
    
    Now the commits on the upstream’s main branch are on your machine. Now you need to create a local branch to track the upstream’s branch:
    
```
$ git checkout -b main upstream/main
Branch 'main' set up to track remote branch 'main' from 'upstream'.
Switched to a new branch 'main'
```
    
    Now you can browse your local copy of the repo:
    
```sh
ls
```
    
- **Store the resulting code in GitHub so you don’t lose work**. After you pull in the lab code, your local repository is “ahead” of the private repository stored on GitHub. Get them into sync:
    
```sh
$ git push origin main
Enumerating objects: 9, done.
....
To github.com:XXX/labs-spr26-YOURUSERNAME.git
 * [new branch]      main -> main
```
    
- **To obtain future labs and changes**: You can check for and merge in changes upstream by typing:
    
```sh
git fetch upstream
git diff <commit_name> upstream/main
git merge upstream/main
```
    
    You should do this periodically. And we will remind you to fetch upstream on Campuswire if we make changes/bug-fixes to the labs.
    
    (Above, `commit_name` is the name of the former head of `upstream/main`. It can be read out after you type `git fetch`.)
    
- **Examining git history**: It’s often helpful to view/browse git history. GitHub can help with this (visit `https://github.com/XXX/labs-spr26-YOURUSERNAME`, and click on the most recent commit id), but of course it can only display commits that are pushed to GitHub. To look at your local repository’s state, you can use `git log`. `git log -p` is particularly handy (displays the differences introduced by every commit).
### Saving changes while you are working on labs

As you modify the skeleton files to complete the labs, you should frequently save your work to protect against laptop failures and other unforeseen troubles, and to create “known good” states. You save the changes by first “committing” them to your local lab repo and then “pushing” those changes to the repo stored on github.com

```sh
git commit -am "saving my changes"
git push origin
```

**Note that whenever you add a new file, you need to manually tell git to “track it”. Otherwise, the file will not be committed by `git commit`.** Make git track a new file by typing:

```sh
git add <my-new-file>
```

After you’ve pushed your changes by typing `git push origin`, they are safely stored on github.com. Even if your laptop catches on fire in the future, those pushed changes can still be retrieved. However, you must remember that doing `git commit` by itself does not save your changes on github.com (it only saves your changes locally). So, don’t forget to type `git push origin`.

To see if your local repo is up-to-date with your origin repo on github.com and vice versa, type `git status`.

### Git FAQ
- **What message should I fill in for git commit -am “message”**?
    The “message” can be any string. But we ask you to leave something descriptive. In the future, when you check your git logs, this message helps you recall what you did for this commit.
- **How can I change a message if it’s already pushed to GitHub?**
    You _can’t_ do this safely. If you want to put another message on top of a previous commit, create an empty commit with your new message:
```sh
git commit --allow-empty -m "<new msg>"
```
    
- **I got an error message `Fatal: Not a git repository`**.
    This means you are typing git commands outside the directory containing your git repository. You need to get back to the `cs5600` directory that you created when you cloned above.
- **Can I edit files through GitHub.com?**
    Do **not** do that this semester. **Super dangerous.** Please only use GitHub.com for read-only access, i.e. checking if all your changes have been pushed to your remote repository.
- **When I do `git fetch`, I got an error `Repository not found`**
    Check the repository address, there should be no quotes (“) or angle brackets (< >). The lab instructions use quotes or angle brackets to mark a placeholder for your GitHub username. If `git fetch upstream` fails, then check the upstream address by typing `git remote -v`. To edit your upstream address, remove it first by typing `git remote remove upstream`, and then add it back with `git remote add`.
- **“The connection timed out” (or problems cloning, or problems with SSH keys)**.
    Check if your firewall is blocking port 22, and open port 22 if it is blocked. You can use your favorite search engine to figure out how to do this.
    
## Setting up the development environment

The labs for this class are intended to be run on a Linux machine. To help make our environments uniform, we will use [Docker](https://docker.com/). The Docker approach to virtualization lets you run a minimal CS5600 environment, including the intended version of Linux, on a Mac OS X, Windows, or Linux, computer, without the overhead of a full virtual machine like [VMware Workstation](https://www.vmware.com/products/workstation-player.html), [VMware Fusion](https://www.vmware.com/products/fusion.html), or [VirtualBox](https://www.virtualbox.org/).

Docker has many advantages. It can start, stop, and edit virtual machines very quickly. In addition, each virtual machine is small, and ocupies little space on your machine. Also, Docker makes it easy to edit your code in your home environment using your preferred IDE or editor, but compile and run it on a Linux host. The disadvantage of Docker, compared to a traditional virtual machine, is that it’s less user-friendly: you have to type strange commands to get it working, and you have to run all programs exclusively in the terminal (no graphical environments).

### Creating CS5600 Docker
We assume that you have cloned your XXX repository into your local machine, as described [earlier](https://XXX).

To build your Docker environment:
1. Launch Docker. On Mac and Windows, there should be a visible whale icon in the notification area (on Mac, the status menu in the upper right toolbar; on Windows, the system tray in the lower right of the screen).
2. In a terminal, change into your `cs5600` directory, and then the `docker`subdirectory, for example:
```sh
cd ~/cs5600  # this is where you cloned the repository earlier
cd docker
```
3. Run the following command. It will take a while, up to ten minutes:
```sh
./cs5600-build-docker
```
    
We may need to change the Docker image during the semester. If we do, we will update `Dockerfile`s in the labs repository. You will update your repository to get the latest `Dockerfile`, then re-run the `./cs5600-build-docker` command from Step 3. However, later runs will in general be faster since they’ll take advantage of your previous work.

### Running CS5600 Docker

Our lab repository contains a script, `cs5600-run-docker`, that provides good arguments and boots Docker into a view of the current directory. This script, too, may be updated throughout the semester.

Here’s an example of running CS5600 Docker on a Mac OS X x86 host (on Windows, you may again need to run `bash` first). At first, `uname` (a program that prints the name of the currently running operating system) reports `Darwin`. But after `./cs5600-run-docker` connects the terminal to a Linux virtual machine, `uname` reports `Linux`. At the end of the example, `exit` quits the Docker environment and returns the terminal to Mac OS X.

```bash

$ cd ~/cs5600
$ uname
Darwin
$ uname -a
Darwin Mike-MacBook-Pro.local 22.1.0 Darwin Kernel Version 22.1.0: Sun Oct  9 20:14:54 PDT 2022; root:xnu-8792.41.9~2/RELEASE_X86_64 x86_64
$ ./cs5600-run-docker
user@f3a862301b38:~/cs5600-labs$ uname
Linux
user@f3a862301b38:~/cs5600-labs$ uname -a
Linux f3a862301b38 5.15.49-linuxkit #1 SMP Tue Sep 13 07:51:46 UTC 2022 x86_64 x86_64 x86_64 GNU/Linux
user@f3a862301b38:~/cs5600-labs$ ls
cs5600-run-docker  docker  lab1  README.md
user@f3a862301b38:~/cs5600-labs$ exit
logout
```

A prompt like `user@f3a862301b38:~$` means that your terminal is connected to the virtual machine (VM). The `f3a862301b38` is a unique identifier for this running VM. You can execute any Linux commands you want. To escape from the VM, type Control-D or run the `exit` command.

The script assumes your Docker container is named `cs5600:amd64` or `cs5600:arm64`, as it was above.

Now you can edit your code on your host, and compile and run it inside Docker.

## Fallback: Khoury or Cloud Servers

If you cannot use your own laptop for some reason, please get in touch with [Adrienne](mailto:XXX), and we can discuss fallback strategies.

## Acknowledgements

Much of this setup is based on Mike Walfish’s CS202 [setup instructions](https://cs.nyu.edu/~mwalfish/classes/25sp/labs/lab2.html). The Docker setup and portions of this writeup were borrowed from [Harvard’s CS61](https://cs61.seas.harvard.edu/). Portions of this writeup were also borrowed from [Jinyang Li’s CS201](http://www.news.cs.nyu.edu/~jinyang/fa18-cso), and [Aurojit Panda’s 3033](https://cs.nyu.edu/courses/fall18/CSCI-GA.3033-002/).