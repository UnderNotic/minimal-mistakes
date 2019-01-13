---
title: "Keeping node applications up to date by syncing with github repo"
excerpt: "Lightweight automated CI pipe aka Poor's man CI"
header:
 teaser:
tags: 
  - nodejs
  - node
  - javascript
  - raspberrypi
  - raspberry
  - ci
  - github
--- 

## The need
Ever wanted to have your running nodejs app synced with github repository?
Let say you have rasperry pi that is running your node app.
Probably You don't want to pull the changes manually everytime just to see updates in your app? It could be great to have some kind of automated deployment, so your app is running up to date source code from github repository.  
So I face this and tried solving it with teamcity and jenkins but those were to heavy for raspberry so I decided to write something more lightweight and today I will share it with You :).

## Prerequirements
There is one and only requirement to set it up (of course outside of github repo with nodejs app):
-  Public IP address 

It's needed because github requires an address that is visible from Internet, not only from behind your router(NAT).

## Overview
To make it work:
- Clone [the repo](https://github.com/UnderNotic/auto-deploy-raspberrypi){: .btn .btn--primary}{:target="_blank"} to raspberry pi or whatever device on which, nodejs app runs
- Setup password needed for github hooks
- Setup your router so it will forward internet network traffic to correct machine behind the router (in your NAT)
- Create github hook in your app repo so on push it will send a signal to koa webserver listening fo it
- Run your node app using pm2/nodemon with file watch enabled so on github push app will be automatically reloaded

## Setting up github hooks
To configure github hooks, usually some work needs to be done on router side.
Our nodejs app will be informed, that repo has been updated
by post request send from github.
To make that happend:
- github needs to know what is `payload url`, which is location of the server
- our server needs to be avaialable from Internet, therafore this requires unblocking port (5000 by default) on local router (router forwarding)  
![image-center](/assets/images/keeping-node-applications-up-to-date-by-syncing-with-github-repo/new_forwarding.png){: .align-center }{:style="width: 100%"}
- our server needs to be avaialable from Internet, therafore this requires unblocking port on local router (router forwarding)   

![image-center](/assets/images/keeping-node-applications-up-to-date-by-syncing-with-github-repo/virtual-servers.png){: .align-center }{:style="width: 100%"}
- 

- 
![image-center](/assets/images/keeping-node-applications-up-to-date-by-syncing-with-github-repo/webhook.png){: .align-center }{:style="width: 100%"}
- place your secret in txt file called secret.txt and place it in root of git-sync project
- 

## Running git-sync

Cloned repo can be run like this:
```bash
node index -p 3000 -r repo1,repo2 -w ~/mine_workspace/
```
Meaning that repos in following directories will be synced:
- ~/mine_workspace/repo1
- ~/mine_workspace/repo2

Paramerts are (in order):
- p = port (default: 5000)
- r = comma seperated repositories names
- repository directory (default: ~/projects/)

To ensure git-sync is running enter localhost on specified port:
![image-center](/assets/images/keeping-node-applications-up-to-date-by-syncing-with-github-repo/package.png){: .align-center }{:style="width: 70%"}

## Note on the code
If You'r interested in details, this is what exactly happens on every code change:
    - github sends post request with metadata 
    - to which repo new code was just pushed
    - who made the push
    - repo name is used to check if repo should be managed by `auto deploy???`
    - tokens from request are checked using local secret to ensure reques was done by github
    - repo code is reset - every dirty change is reseted
    - repo code is cleaned - every added local file is removed
    - new code is pulled 
    - npm install all dependencies for production environment

```javascript
router.post('/payload', async ctx => {
    let repo = ctx.request.body.repository.name;
    console.log(`${ctx.request.body.sender.login} just pushed to ${repo}`);

    if (allowedRepos.indexOf(repo) === -1 && await verifySignature(ctx.request, secretTokenPromise)) {
        console.error(`${repo} is not allowed`);
    } else {
        await execAsync(`git -C ~/projects/${repo} reset --hard`);
        await execAsync(`git -C ~/projects/${repo} clean -df`);
        await execAsync(`git -C ~/projects/${repo} pull -f`);
        await execAsync(`npm -C ~/projects/${repo} install --production`);
    }
    ctx.status = 200;
});
```

## Last but not least - automatic server restarts - PM2
By adding pm2 into the equation, You can have automatic server restarts every time code for app changes in other words pushing code to github will automatically refresh server to use the newest code, this is additionaly done gracefully without any hiccups.

Once there is code in `~/mine_workspace/repo1` create pm service 2

This service will starts on system startup and by default will watch directory for any code changes.
If pm2 is too heavy, use nodemon as a replacement.

Try it out and save your time :)
