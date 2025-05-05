+++ 
draft = false
date = 2025-05-05T17:32:49+01:00
title = "Creating a static website using Hugo and hosting it on Github"
description = ""
slug = ""
authors = []
tags = []
categories = []
externalLink = ""
series = []
+++

I'm going to lay it out here, the steps I took to create this website and then host it on github, as the official [guide](https://gohugo.io/hosting-and-deployment/hosting-on-github/) isn't hugely clear. I am going to split this post into two sections: [creation of the site](#1), and then [deploying it to github](#2).

# Creating the site # {#1}
The first step is to install [Hugo](https://gohugo.io/getting-started/installing/).
I'm running Linux so I used the following in the command line:
```
sudo apt-get install hugo
```
Now, you can go ahead and create a website using:
```
hugo new site <website-name> # replace <website-name> to something of your choice
```
For the moment it's just a blank canvas so to change that a theme needs to be chosen.
Navigating to [themes](https://themes.gohugo.io/) you can see a large collection of designs to choose from.

After deciding on the theme you want, click on it. Click on the download button and you will be sent to its github page. I used [coder](https://github.com/luizdepra/hugo-coder). 

Download the theme by performing the following:
```
cd <website-name>

git init
git submodule add https://github.com/luizdepra/hugo-coder themes/coder
cp themes/coder/exampleSite/hugo.toml config.toml
```
Open up `config.toml` in a text editor to customise the theme and start making alterations/additions eg. title

Content can be created by using:
```
hugo new posts/my-first-post.md
```
Opening up `my-first-post.md` will show something like this:
```
---
title: "My First Post"
date: 2019-03-26T08:47:11+01:00
draft: true
---
```
Once you finish a post, update the header of the post to say draft: false, else your content will not get deployed!

Now to see the result - in the terminal, type:
```
hugo server # -D drafts enabled if you haven't updated your post headers
```
Navigate to your new site at http://localhost:1313/.

Make any other configurations or changes and your site will reflect them straight away.

# Deploying the site # {#2}
The first thing to do is create two new repositories in your Github account. The first `<YOUR-PROJECT>` to host your generated content, and the other `<USERNAME>.github.io` containing the fully rendered version of the website. Ensure that they're both empty. You can host on a User Page or on a Project Page; as this is my personal blog/portfolio, I went with the former.

```
git clone <YOUR-PROJECT-URL> && cd <YOUR-PROJECT>
```
Paste your existing Hugo project into a new local `<YOUR-PROJECT>` repository. Make sure your website works locally (hugo server).
Once you are happy with the results:

    Press Ctrl+C to kill the server
    Before proceeding run rm -rf public to completely remove the public directory
  
Open up `config.toml` and replace the baseURL from `"https://example.org/"` or `"http://localhost:1313/"` to the domain of your new website: `"at8865.github.io"` (in my case). 

Add a submodule so that the /public directory will have the github host repo as its origin:
```
git submodule add -b master git@github.com:<USERNAME>/<USERNAME>.github.io.git public # change <USERNAME> and to https if preffered
```
Create a `deploy.sh` script and paste the following in it:
```
#!/bin/sh

# If a command fails then the deploy stops
set -e

printf "\033[0;32mDeploying updates to GitHub...\033[0m\n"

# Build the project.
hugo # if using a theme, replace with `hugo -t <YOURTHEME>`

# Go To Public folder
cd public

# Add changes to git.
git add .

# Commit changes.
msg="rebuilding site $(date)"
if [ -n "$*" ]; then
	msg="$*"
fi
git commit -m "$msg"

# Push source and build repos.
git push origin master
```
In the command line, make it executable with `chmod +x deploy.sh`. Then run `./deploy.sh "Your optional commit message"`.
Finally, commit the changes to `<YOUR-PROJECT>` repository as well. And voila, the website should be up and running at `https://<USERNAME>.github.io`!