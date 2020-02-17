---
title: Hexo
date: 2017-09-02 11:17:15
tags: 
---
Welcome to the Hexo documentation. If you encounter any problems when using Hexo, have a look at the [troubleshooting guide](https://hexo.io/docs/troubleshooting), raise an issue on [GitHub](https://github.com/hexojs/hexo/issues) or start a topic on the [Google Group](https://groups.google.com/group/hexo).

## What is Hexo?

Hexo is a fast, simple and powerful blog framework. You write posts in [Markdown](http://daringfireball.net/projects/markdown/) (or other markup languages) and Hexo generates static files with a beautiful theme in seconds.

<!-- more -->

## Installation

It only takes a few minutes to set up Hexo. If you encounter a problem and can’t find the solution here, please [submit a GitHub issue](https://github.com/hexojs/hexo/issues) and we’ll help.

### Requirements

Installing Hexo is quite easy and only requires the following beforehand:

- [Node.js](http://nodejs.org/) (Should be at least Node.js 8.10, recommends 10.0 or higher)
- [Git](http://git-scm.com/)

If your computer already has these, congratulations! You can skip to the [Hexo installation](https://hexo.io/docs/index.html#Install-Hexo) step.

If not, please follow the following instructions to install all the requirements.

### Install Git

- Windows: Download & install [git](https://git-scm.com/download/win).
- Mac: Install it with [Homebrew](https://brew.sh/), [MacPorts](http://www.macports.org/) or [installer](http://sourceforge.net/projects/git-osx-installer/).
- Linux (Ubuntu, Debian): `sudo apt-get install git-core`
- Linux (Fedora, Red Hat, CentOS): `sudo yum install git-core`

> For Mac users
>
> You may encounter some problems when compiling. Please install Xcode from App Store first. After Xcode is installed, open Xcode and go to **Preferences -> Download -> Command Line Tools -> Install** to install command line tools.

### Install Node.js

Node.js provides [official installer](https://nodejs.org/en/download/) for most platforms.

Alternative installation methods:

- Windows: Install it with [nvs](https://github.com/jasongin/nvs/) (recommended) or [nvm](https://github.com/nvm-sh/nvm).
- Mac: Install it with [Homebrew](https://brew.sh/) or [MacPorts](http://www.macports.org/).
- Linux (DEB/RPM-based): Install it with [NodeSource](https://github.com/nodesource/distributions).
- Others: Install it through respective package manager. Refer to [the guide](https://nodejs.org/en/download/package-manager/) provided by Node.js.

nvs is also recommended for Mac and Linux to avoid possible permission issue.

> Windows
>
> If you use the official installer, make sure **Add to PATH** is checked (it’s checked by default).

> Mac / Linux
>
> If you encounter `EACCES` permission error when trying to install Hexo, please follow [the workaround](https://docs.npmjs.com/resolving-eacces-permissions-errors-when-installing-packages-globally) provided by npmjs; overriding with root/sudo is highly discouraged.

### Install Hexo

Once all the requirements are installed, you can install Hexo with npm:

```
$ npm install -g hexo-cli
```

### Advanced installation and usage

Advanced users may prefer to install and use `hexo` package instead.

```
$ npm install hexo
```

Once installed, you can run Hexo in two ways:

1. `npx hexo <command>`

2. Linux users can set relative path of `node_modules/` folder:

   ```
   echo 'PATH="$PATH:./node_modules/.bin"' >> ~/.profile
   ```