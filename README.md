## Introduction

This is the source code for my blog: [https://ntflc.com](https://ntflc.com), which is powered by [Hexo](https://hexo.io).

## How to build

You need Node.js and Hexo installed. I only test the build on Node.js `v18.12.1`, npm `8.19.2`, and Hexo `7.0.0`.

Steps to compile this blog:

``` sh
# Clone source code
git clone git@github.com:ntflc/hexo-source.git
# Go to folder of source code
cd hexo-source
# Install npm packages
npm install
# Generate static files
hexo generate
# Deploy to GitHub
hexo deploy
```

## Common commands

```sh
# Create a new post
hexo new "Name-of-New-Post"
# Create a new page
hexo new page "Name-of-New-Page"
# Generate static files
hexo generate
# Start local server
hexo server
# Deploy to GitHub
hexo deploy
```
