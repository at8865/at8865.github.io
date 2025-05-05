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

The recommended way to deploy a Hugo site to GitHub Pages is using GitHub Actions. This automates the build and deployment process whenever you push changes to your repository.

1.  **Create the GitHub Repository:**
    Create a new **public** repository in GitHub named `<USERNAME>.github.io`, replacing `<USERNAME>` with your actual GitHub username. This specific naming convention tells GitHub you want to host a User Page.

2.  **Push Your Hugo Site Source Code:**
    Initialize a git repository in your local Hugo project folder (`<website-name>`) if you haven't already. Add all your project files (the `config.toml`, `content/`, `themes/`, `static/`, etc., **but not** the generated `public/` folder) to git. Set your new `<USERNAME>.github.io` repository as the remote origin and push your code to the `master` (or `main`) branch.

    ```bash
    cd <website-name> # Navigate to your Hugo project directory
    # If you haven't already initialized git:
    git init
    git remote add origin https://github.com/<USERNAME>/<USERNAME>.github.io.git # Replace <USERNAME>
    # Add all your files
    git add .
    # Commit the files
    git commit -m "Initial commit of Hugo site source"
    # Push to GitHub (use -u to set upstream for the first push)
    git push -u origin master # Or 'main' if that's your default branch
    ```

3.  **Create the Workflow File:**
    In your local project, create the directory path `.github/workflows/`. Inside the `workflows` directory, create a file named `hugo.yml` (or any other `.yml` name). Paste the following workflow content into this file:

    ```yaml
    # Sample workflow for building and deploying a Hugo site to GitHub Pages
    name: Deploy Hugo site to Pages

    on:
      # Runs on pushes targeting the default branch
      push:
        branches: ["master"] # Or "main" if that's your default branch

      # Allows you to run this workflow manually from the Actions tab
      workflow_dispatch:

    # Sets permissions of the GITHUB_TOKEN to allow deployment to GitHub Pages
    permissions:
      contents: read
      pages: write
      id-token: write

    # Allow only one concurrent deployment, skipping runs queued between the run in-progress and latest queued.
    # However, do NOT cancel in-progress runs as we want to allow these production deployments to complete.
    concurrency:
      group: "pages"
      cancel-in-progress: false

    # Default to bash
    defaults:
      run:
        shell: bash

    jobs:
      # Build job
      build:
        runs-on: ubuntu-latest
        env:
          HUGO_VERSION: 0.128.0 # Specify your desired Hugo version
        steps:
          - name: Install Hugo CLI
            run: |
              wget -O ${{ runner.temp }}/hugo.deb https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.deb \
              && sudo dpkg -i ${{ runner.temp }}/hugo.deb
          - name: Install Dart Sass # Optional: If your theme uses Sass
            run: sudo snap install dart-sass
          - name: Checkout
            uses: actions/checkout@v4
            with:
              submodules: recursive # Fetch theme submodules
          # Optional: Setup Node if using npm packages (e.g., for PostCSS)
          # - name: Setup Node
          #   uses: actions/setup-node@v4
          #   with:
          #     node-version: '20' # Specify Node version
          #     cache: 'npm'
          # - name: Install Node.js dependencies
          #   run: "[[ -f package-lock.json || -f npm-shrinkwrap.json ]] && npm ci || true"
          - name: Build with Hugo
            env:
              HUGO_ENVIRONMENT: production
            run: |
              hugo \
                --minify \
                --baseURL "/" # BaseURL is root for user pages
          - name: Upload artifact
            uses: actions/upload-pages-artifact@v3
            with:
              path: ./public # Hugo builds to 'public' directory by default

      # Deployment job
      deploy:
        environment:
          name: github-pages
          url: ${{ steps.deployment.outputs.page_url }}
        runs-on: ubuntu-latest
        needs: build
        steps:
          - name: Deploy to GitHub Pages
            id: deployment
            uses: actions/deploy-pages@v4
    ```

    Commit this new workflow file and push it to GitHub:
    ```bash
    git add .github/workflows/hugo.yml
    git commit -m "Add GitHub Actions workflow for Hugo deployment"
    git push
    ```

4.  **Configure GitHub Pages Settings:**
    Go to your repository settings on GitHub (`https://github.com/<USERNAME>/<USERNAME>.github.io/settings/pages`). Under "Build and deployment", set the "Source" to **GitHub Actions**.

5.  **Automatic Deployment:**
    Now, whenever you push changes (like new posts or theme updates) to your `master` (or `main`) branch, the GitHub Action will automatically run, build your Hugo site, and deploy the contents of the `public` directory to `https://<USERNAME>.github.io`. You can monitor the progress in the "Actions" tab of your repository.