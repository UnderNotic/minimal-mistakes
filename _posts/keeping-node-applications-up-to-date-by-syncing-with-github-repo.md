---
title: "Keeping node applications up to date by syncing with github repo aka Poor's man CI"
excerpt: "Lightweight automated CI pipe"
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
Probably You don't want to pull the changes everytime just to see updated app running? It could be great to have some kind of automated deployment, so your app is running up to date source code from github repository.  
So I face this and tried solving it with teamcity and jenkins but those were to heavy for raspberry so I decided to write something more lightweight and today I will share it with You :).


## Prerequirements
There is one and only requirement to set it up (of course outside of nodejs app and cloned github repo :D):
-  Public IP address 

It's needed because github needs an address that is visible from Internet, not only from behind your router(NAT).


## The Setup
To make it work:
- Clone [the repo](https://github.com/UnderNotic/auto-deploy-raspberrypi){: .btn .btn--primary}{:target="_blank"} to raspberry pi or whatever device on which, nodejs app runs
- Setup password needed for github hooks
- Setup your router so it will forward internet network traffic to correct machine behind the router (in your NAT)
- Create github hook in your app repo so on push it will send a signal to koa webserver listening fo it
- Run your node app using pm2 with file watch enabled so on github push app will be automatically reloaded

## What's inside

Cloned repo can be run like this:
```bash
node index 3000 repo1 repo2 ~/mine_workspace/
```
Meaning that repos in following directories will be synced:
- ~/mine_workspace/repo1
- ~/mine_workspace/repo2

Paramerts are (in order):
- port (default: 5000)
- synced repos (default: test-repo)
- repository directory (default: ~/projects/)


## Github hooks
To configure github hooks, usually some work needs to be done on router side.
Our nodejs app will be informed, that repo has been updated
by post request send from github. 
To make that happend:
- github needs to know what is `payload url`, which is location of the server
- our server needs to be avaialable from Internet, therafore this requires unblocking port on loca router (router forwarding)   
![image-center](/assets/images/keeping-node-applications-up-to-date-by-syncing-with-github-repo/new_forwarding.png){: .align-center }{:style="width: 100%"}
![image-center](/assets/images/keeping-node-applications-up-to-date-by-syncing-with-github-repo/virtual-servers.png){: .align-center }{:style="width: 100%"}

![image-center](/assets/images/keeping-node-applications-up-to-date-by-syncing-with-github-repo/package.png){: .align-center }{:style="width: 90%"}

![image-center](/assets/images/keeping-node-applications-up-to-date-by-syncing-with-github-repo/webhook.png){: .align-center }{:style="width: 100%"}

## Router forwarding

## Note on the code
If You'r interested in details, this is what exactly happens on every code change:
- github sends post request with metadata 
    - to which repo new code was just pushed
    - who made the push
    - repo name is used to check if repo should be managed by `auto deploy???`
    - tokens ???
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


## Last but not least - PM2
By adding pm2 into the equation, You can have automatic server restarts every time code for app changes in other words pushing code to github will automatically refresh server to use the newest code, this is additionaly done gracefully without any hikups.

Once there is code in `~/mine_workspace/repo1` create pm service 2

This service will starts on system startup and by default will watch directory for any code changes.

Try it out and save your time :)

