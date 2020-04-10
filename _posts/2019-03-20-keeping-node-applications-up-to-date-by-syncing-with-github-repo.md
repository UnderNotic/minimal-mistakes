---
title: "Keeping node applications up to date by syncing with github repo"
excerpt: "Lightweight automated CI pipe aka Poor's man CI"
header:
 teaser: "assets/images/keeping-node-applications-up-to-date-by-syncing-with-github-repo/teaser.png"
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

> Ever wanted to have your nodejs app synced with github repository at runtime?

Let say you have raspberry pi that is running your node app.
Probably You don't want to pull the changes manually every time just to see updates in your app? It could be great to have some kind of automated deployment, so your app is running up to date source code from github repository.
  
I faced this and tried solving it with teamcity and jenkins but those were to heavy and complicated for raspberry so I decided to write something more lightweight and today I will share it with You :).

## One thing that is required

To set it up, github has to know what to call, meaning server with node application needs to be exposed to public internet.

If that's already the case, great, if not then it has to be exposed.
There are two options:
- fully manual that requires public IP and enabled forwarding of port 5000 on router, you can read about it [here.]({{ page.url | absolute_url }}exposing-local-server-to-the-public-internet/){:target="_blank"}
- tools like ngrok, which do all the heavy lifting for you

Eventually this public url will be used by github to send post request with information, that given repo has been updated.

## Github repo setup

To make it work:
- Clone [the repo](https://github.com/UnderNotic/auto-deploy-raspberrypi){: .btn .btn--primary}{:target="_blank"} to raspberry pi or whatever device on which, nodejs app runs
- Setup password needed for github hooks and put it to secret.txt file placed in the root of cloned repo
- Create github hook in your app repo so on push it will send a signal to koa webserver listening for it
![image-center](/assets/images/keeping-node-applications-up-to-date-by-syncing-with-github-repo/webhook.png){: .align-center }{:style="width: 100%"}

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
- repo name is used to check if repo should be managed by `git-sync`
- tokens from request are checked using local secret to ensure request was done by github
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

By adding pm2 into the equation, You can have automatic server restarts every time code for app changes. In other words pushing code to github will automatically refresh server to use the newest code and this is done gracefully without any hiccups.

PM2 service will start on system startup and by default will watch directory for any code changes.
If pm2 is too heavy, use nodemon as a replacement.

Try it out and save your time :)
