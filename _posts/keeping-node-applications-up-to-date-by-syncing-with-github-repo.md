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

###
Ever wanted to have your running nodejs app synced with github repository? 
Let say you have rasperry pi that is running your node app.
Probably You don't want to pull the changes everytime just to see updated app running? It could be great to have some kind of automated deployment, so your app is running up to date source code from github repository.  


### Prerequirements

There is one and only requirement to set it up (ofcourse outside of nodejs app and github repo :D):
-  Public Ip address 

It's needed because github needs an address that is visible from Internet, not only from behind your router(NAT).







### PM2
By adding pm2 into the equation




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