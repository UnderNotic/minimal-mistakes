---
title: "Keeping node applications up to date by syncing with github repo"
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

### The need
Ever wanted to have your running nodejs app synced with github repository? 
Let say you have rasperry pi that is running your node app.
Probably You don't want to pull the changes everytime just to see updated app running? It could be great to have some kind of automated deployment, so your app is running up to date source code from github repository.  
So I face this and tried solving it with teamcity and jenkins but those were to heavy for raspberry so I decided to write something more lightweight and today I will share it with You :).


### Prerequirements
There is one and only requirement to set it up (ofcourse outside of nodejs app and github repo :D):
-  Public Ip address 

It's needed because github needs an address that is visible from Internet, not only from behind your router(NAT).


### The Setup
To make it work:
- clone mine repo to raspberry pi or whatever device on which, nodejs app runs
- Setup your router so it will forward internet network traffic to correct machine behind the router (in your NAT)
- Create github hook in your app repo so on push it will send a signal to koa webserver listening fo it
- Run your node app using pm2 with file watch enabled so on github push app will be automatically reloaded

### What inside
Syncing app can be run like this:
```bash
node index 3000 mine-repo1;mine-repo2 ~/mine_workspace/
```
Meaning that repos in following directories will be synced:
- ~/mine_workspace/mine_repo1
- ~/mine_workspace/mine_repo2

Paramerts are (in order):
- port (default: 5000)
- synced repos (default: test-repo)
- repository directory (default: ~/projects/)






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


### PM2
By adding pm2 into the equation

