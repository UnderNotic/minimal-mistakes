---
title: "Running C# in REPL using scriptcs"
excerpt: "C# repl everywhere"
header:
 teaser: "assets/images/scripting-in-c-sharp.png"
tags: 
  - scripting
  - repl
  - scriptcs
  - vscode
  - .net
  - .netcore
  - csharp
  - linux
--- 

## C# REPL

Did You ever wanted to run C# code in a scripty inline way to check if code will execute the way you want?

This could be especially useful when you don't have powerful visual studio with repl(interactive) and You are writing C# in something lightweight like `vs code`.

I decided to write this post because installing scriptcs is cumbersome, most of the guides/docs out there don't help much.

## On Windows

On Windows it's pretty straightforward:  
- Install `chocolatey`, in powershell type: 
```powershell
Set-ExecutionPolicy AllSigned; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
```
- ...or in command prompt:
```powershell
@"%SystemRoot%\System32\WindowsPowerShell\v1.0\powershell.exe" -NoProfile -InputFormat None -ExecutionPolicy Bypass -Command "iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))" && SET "PATH=%PATH%;%ALLUSERSPROFILE%\chocolatey\bin"
```

- Install scriptcs: 
```
choco install scriptcs
```
- It's added to path automatically, easy mode.


## On Linux

Troubles begin on linux. Since you don't have chocolatey there and striptcs doesn't provide any deb package.
It took me humongous amount o time to figure out how to make it work. I had to check on travis how project is build on linux.

For sake of this I'm using mint 18, for ubuntu should be the same.

Prerequisites are:

- `mono` [(get it here)](http://www.mono-project.com/download/#download-lin){:target="_blank"}

- `msbuild` (xbuild from mono didn't work for me).

- install msbuild:  
```bash
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 3FA7E0328081BFF6A14DA29AA6A19B38D3D831EF
echo "deb http://download.mono-project.com/repo/ubuntu xenial main" | sudo tee /etc/apt/sources.list.d/mono-official.list
sudo apt-get -qq update
sudo apt-get install msbuild
```

- checkout scriptcs sources and cd into:
```bash
git clone https://github.com/scriptcs/scriptcs.git
cd scriptcs
```

- restore nuget packages
```bash
mono ./.nuget/NuGet.exe restore ./.nuget/packages.config -PackagesDirectory ./packages
mono ./.nuget/NuGet.exe restore ./ScriptCs.sln
```

- compile it using msbuild
```bash
msbuild ./ScriptCs.sln /property:Configuration=Release /nologo /verbosity:normal
```

- add bash script, that will execute compiled exe using mono
```bash
  sudo mkdir /usr/share/scriptcs && cd "$_"
  ## copy files from build output directory (/bin/release/) to current directory
  sudo touch scriptcs
  chmod +x scriptcs
  sudo nano scriptcs 
  ## copy & paste 
  #!/usr/bin/env bash 
  mono "/usr/share/scriptcs/scriptcs.exe"
```

- create symlink to add scriptcs to path
```bash
ln -s {path to bash script} /usr/bin/scriptcs
```

## Code Runner 
Just type `scriptcs` in terminal and here you go.

If you want something more neat use it with `vs code` and `Code Runner` extension, which is using scriptcs to run selected code.  
Go grab it, select C# code in vs code, hit ctrl + alt + n and enjoy output from scriptcs (remember to print out what you want to see).

If you are looking for similar thing but for `js` check `Quokka.js`.

Cheers!
