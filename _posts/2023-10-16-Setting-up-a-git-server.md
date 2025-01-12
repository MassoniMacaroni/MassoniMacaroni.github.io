---
layout: post
title: "Setting up a git server"
categories: Posts
tags: [Git, Linux, version control, servers, infrastructure]
image: assets/img/Setting-up-a-git-server/GitServerBanner.png

---

I'm currrently, in the process of gaining as many of the skills to get a DevOps or SRE related job and figured "What's one of the most important things for any development environment?" - version control. I know the basics of git like push/pull, branching, and merging but I'm certainly no expert and thats why I've decided I'm going to try set up a Git server. Importantly this isn't intended to be a guide but instead a recount of my experiene of the process. 

## What is Git?
Git is a widely used version control system that helps track changes in source code during software development. Created by Linus Torvalds in 2005, it's known for its distributed architecture, where each developer's working copy of the code acts as a repository with a complete history of changes. I think all of Atlassian's content on Git is pretty good and worth a read if you're learning about Git [What is Git?](https://www.atlassian.com/git/tutorials/what-is-git).

## But what does a Git server do?
A basic Git server hosts Git repositories and facilitates collaboration by allowing developers to share, push, and pull code changes. It keeps track of the version history of projects, manages user permissions, and provides a centralized location for the authoritative versions of projects while maintaining Git's distributed version control advantages.

## Setting up the server.
I'm going to be following most of the information provided on git-scm.com specifically chapters 4 & 5 of the [Pro Git Book](https://git-scm.com/book/en/v2). Because at a glance it seems like the easiest method to setup (and I'm only learning) I'm going to use SSH [Git Server Protocols](https://git-scm.com/book/en/v2/Git-on-the-Server-The-Protocols). I'm setting this up on an almost fresh Centos 8 machine so first I'll need to install git.

```
yum install git
```

The documentation mentions next a bare repository is required — a repository that doesn’t contain a working directory and has steps on how to create it. I was unsure what the difference is between a newly initialised repo and a bare repo was and found this was a good source [What is a bare repo](https://blog.opstree.com/2022/06/21/what-is-a-bare-git-repository/). Next I created a bare repo.

```
mkdir /srv/git
# Navigate to /srv/git
git init --bare gitserver
```

Now from another computer with SSH access to the server git is on I am able to clone the bare repo. Note: I believe for easier readability it would've been better if I named my repo gitserver.git. Users will require read access to clone the repo and write access to push to the repo.

```
git clone root@192.168.1.107:/srv/git/gitserver 
```

With SSH access more complex permissions can be handled with file system privileges which I am not using here for simplicities sake.

### Generating your SSH Public Key
We need to verify if we already have an SSH key and if we don't we'll create one. To see if your user already has an SSH key navigate to ~/.ssh
```
cd ~/.ssh
ls
```
In my case this directory did not exist but you're looking for something along the lines of d_dsa or id_rsa and a matching file with a .pub extension. The .pub file is your public key, and the other file is the corresponding private key. As per the the Git pro book I used ssh-keygen to create the required key after creating the ~/.ssh directory that didnt exist.
```
mkdir ~/.ssh
ssh-keygen -o
```
File to save the key: .ssh/id_rsa and enter twice for a blank passphrase.

The documentation mentions each user that does this has to send their pub file which would be "id_rsa.pub" to the server administration (I'm not quite sure where it goes yet).

### Setting up the server
This part outlines setting up SSH access on the server side using the [Authorized keys](https://www.ssh.com/academy/ssh/authorized-keys-file) method for authenticating users. 

First a git user needs to be created and a .ssh directory for the git user.
```
sudo adduser git
su git
cd
mkdir .ssh && chmod 700 .ssh
touch .ssh/authorized_keys && chmod 600 .ssh/authorized_keys
```

Next, we need to add some developer public ssh keys to the authorized_keys file of our git user. Unfortunately I don't have any public keys laying around so I'm going to make another keypair in tmp and use the public key from that.

```
ssh-keygen -o
Enter file in which to save the key (/home/git/.ssh/id_rsa): /tmp/id_rsa.testuser
```
I now have id_rsa.testuser and id_rsa.testuser.pub located in /tmp/ and continuing on with the guide we next need to append the public key to the git users authorized_keys file.

```
cat /tmp/id_rsa.testuser.pub >> ~/.ssh/authorized_keys
```

Aftering appending the public key to authorized_keys the git user can now create another --bare repo but this time users with their public key added can easily read/write to repo as the git user.

## Conclusion
Setting up a very basic Git server has certainly improved on my practical knowledge of Git concepts but there is still a lot to learn. I hope you also found this insightful as I did but remember this certainly isn't anything close to a best practice demonstration or walk-through.

Thanks for reading!


