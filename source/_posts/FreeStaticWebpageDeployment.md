---
title: Free Static Webpage Deployment
date: 2021-01-24 19:05
tags: 
	- Web deployment
	- git
categories: 新手指北
id: freeWebpage
---

I have a number of static websites on hand, including a CV, an anniversary website, a small blog and miniature functional static pages such as QR code generation that need to be deployed on the internet. The best option would undoubtedly be to deploy on a private cloud server, but the functionality of these sites does not justify renting an expensive server for them. In this context, here is a solution for webpage deployment based on github or bitbucket.

<!-- more -->

# Overview of steps

- Find an available mailbox;
- Sign up for bitbucket or github using your email address;
- Configuring local SSH;
- Create a cloud repository with a specific name;
- Push local repositories to the cloud;
- [Option] Binding personal domains.

# Find an available mailbox

Of course I can use my personal email, but it is tied to my mobile number and I don't want to receive relevant tweets or ads. I only use this email address to register for github or bitbucket for the time being and will not need it again once my account is registered. So I opted for a temporary email address provided by a third party (NO PHONE NUMBER NEEDED), and here are some of the things I gathered and tried:

- [Snapmail](https://www.snapmail.cc/): Always available, but emails will be automatically deleted after 4 hours. I used this email address to successfully sign up for bitbucket and deployed https://abadanniversary2.bitbucket.io. 
- [TempMail](https://temp-mail.org/): It is more widely available, but uses a mailbox for a period of time and does not support customisation., which requires the purchase of additional services at a cost. If you need to change to a new mailbox, you will need to delete the current mailbox.
- [临时邮箱 - 十秒钟内收到邮件](https://linshiyouxiang.net/): A very good temporary email provider, I have used this site to register for many forums that require email logins for temporary use. But the email here doesn't seem to register with bitbucket, at least not for me. Also this email address is prompted with "Your account has been flagged. Because of that, your profile is hidden from github" after registering for github, you can appeal here but given the time cost and the need for temporary use, I've opted out.
- [24mail](http://24mail.chacuo.net/): No registration is required and the mailbox lasts 24 hours, which is longer than a ten-minute mailbox (10 minutes). You can set any mailbox name and change your mailbox at any time. However, after trying, there are some websites that do not support this mailbox, such as github.

> There are other free temporary mailboxes or alternate mailboxes that do not require a mobile number to be registered, which will be added here in the future after testing.

# Sign up for bitbucket or github using your email address

This step should not be difficult, make sure that your github account is not flagged. If it is, please send an email requesting to be unblocked or change your registered email address.

# Configuring local SSH

## 1. Open the git Bash

Open a git Bash command window anywhere and type the following command to access the .ssh directory:

```sh
cd ~/.ssh
```

If the path does not exist, you will need to manually create this folder in `C:\Users\yourAccountName\.ssh`.

## 2. Generate SSH for different accounts

Generate the corresponding ssh with the following command, and run as many lines of the corresponding command as the number of accounts needed for ssh.

```sh
$ ssh-keygen -t rsa -C "theEmailRegisted@email.com" -f "githubForWeb_id_rsa"
$ ssh-keygen -t rsa -C "theEmailRegisted@email.com" -f "bitbucketTemp_id_rsa"
```

## 3. Copy the public key to platforms such as github

- Execute the command `cat ~/.ssh/githubForWeb_id_rsa.pub` and copy the second line to the end to the ssh of github and save it.
- Or you can simply open the file directory and find the corresponding public key file `githubForWeb_id_rsa.pub`, then open the file with an editor and copy all its contents to the SSH settings of your account.

The same operation can be used to add SSH keys for platforms such as bitbucket, gitee, etc.

## 4. Create config file to resolve ssh conflicts

Execute the command `vi config` in the .ssh folder and add the following to the file

```sh
# github
Host myWebOnGithub
HostName github.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/githubForWeb_id_rsa

# bitbucket
Host myWebOnBitbucket.org
HostName bitbucket.org
PreferredAuthentications publickey
IdentityFile ~/.ssh/bitbucketTemp_id_rsa
```

Once the configuration is complete, use `:wq` to save and exit the editor. You can change the Host and HostName as required, and you can just create a new `config` file in the folder and right click to edit it.

## Test

Test each of the following codes separately:

```sh
ssh -T git@myWebOnGithub
```

```sh
ssh -T git@myWebOnBitbucket.org
```

If there is no problem it will say that the login was successful, but it may say something like `but GitHub does not provide shell access`. This does not affect the connection and means that the configuration has been successful.

# Create a cloud repository with a specific name

Create a repository in github named `username.github.io`, e.g. if the username is `AbAdAnniversary`, you need to create a repository named `AbAdAnniversary.github.io`.

If you are using bitbucket, the repository name needs to be `username.bitbucket.io`; the same naming convention applies to platforms such as gitee

# Push local repositories to the cloud

With git installed correctly, start git Bash from the root of your local repository.

- Type `git init` on the command line to turn the folder into a manageable Git repository;

- Then you can add the file via `git add . `(note the "." with spaces in it, "." means that all the directories under this folder are committed. You can also add a file to the cache by `git add filename`);

- [Option] You can then check the current status by using the git status command;

- Then, use the command `git commit -m "write your comments here"` to commit the file to the local repository;

- Connecting to a remote warehouse:

  - Test the corresponding SSH with the command `ssh -T git@myWebOnBitbucket.org`;

  - Linked remote and local repositories now, Enter the following command in the local repository Git Bash:

    `$ git remote add origin git@github.com:username/repositoryName.git` The git address after origin can be obtained via clone from the cloud repository, both SSH and HTTP methods are available. SSH is preferred.

- Once the association is done we can push all the content from the local repository to the remote repository:

  `$ git push -u origin master`

If the remote repository contains files that are not in the local repository, a direct push will report an error. For example, when creating a new repository, "Initialize this repository with a README" is checked, but the local repository does not have this file. Here you can pull the contents of the remote repository locally, merge it and commit it:

- `$ git pull --rebase origin master`
- `$ git push -u origin master`

This can also be pushed through tools such as sourcetree, where we can first clone from the remote repository and then push, while taking care to set the SSH Key for sourcetree to ensure write access.

# [Option] Binding personal domains

## Adding DNS records

Open the console in the Personal Domain Name Service platform you purchased, click on Resolve Domain Name (DNS) and create the following DNS record:

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@main/blog/2021/01/25/sqg9pD.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">The red arrows point out the necessary DNS (First & second), the rest are baidu's and google's webmaster statistics tools</div>
</center>

Note the types of these two DNSs. For the record of type A, fill in @ for Name (here I'm using CloudFlare so I'm filling in my personal domain address) and the ip address for username.github.io for content. This ip can be obtained by `ping username.github.io`. Since my username of github is shimmerjordan, I put shimmerjordan here.

## Adding CNAME file

Create a new CNAME file in the local repository root directory and edit its contents to your personal domain name. In my case, for example, the CNAME file will read blog.shimmerjordan.eu.org. be sure to note that the CNAME file does not have any suffixes. Then push it to the remote repository.

## Go to github settings and bind the domain name

In the repository settings, pull down to the bottom and find GitHub Pages, select the appropriate source branch and fill in the Custom domain with your personal domain name. Finally, check Enforce HTTPS if required and select Theme Chooser.

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@main/blog/2021/01/25/sqRclD.png" width=70%>
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">My Case</div>
</center>

Click save to complete the domain binding.

# Attachment

This is all I have done, and here are the current pages I have successfully deployed by this method:

- [My blog](https://blog.shimmerjordan.eu.org/)

- [My CV](https://cv.shimmerjordan.eu.org/)

- [My grilfriend's CV](https://ruichenzhao.bitbucket.io/)

- [A little surprise for our second anniversary](https://abadanniversary2.bitbucket.io/)

