---
title: Towards a README for this website
modified: Thursday 20th November 2025 14:48:52
date: 2025-11-20
showDate: true
draft: true
showPagination: false
pin: false
categories:
  - documentation
---
# Towards a README for this website

This is for my own future reference and for anyone who might be interested in Obsidian, static websites, and deploying via Git.

## Obsidian

These days, I like to use [Obsidian](https://obsidian.md/) to organize the content of my websites because I already use it for notes. I have a vault for each website, currently [the one you are looking at](https://somethingilearned.today) and my [archive of art and writing](https://metadada.xyz). I could just use any names for folders and files that feels right (so on the other site, the main folders are `quasi-institutions` and `studies`) but because this site is a weblog, I use `posts` as the folder to contain all my posts.


![[attachments/Pasted image 20251120151339.png]]
///caption
What my Obsidian looks like for this page on this site.
///

<!--more-->
### Plugins

If you've never used Obsidian, you should know that you are almost definitely going to want to install some plugins to customize it. Once these are all set up, I never think about them again, and so documenting it here is how I stop myself from reverse engineering my own system:

#### Git 

This is the single most important and essential plugin. Not only does it store and version control my files but it is how I publish my website (using **GitHub Actions** -- more on that below). But see that up arrow in a circle, above where it says "vault backup"? When I push that it publishes my site.

#### Templater

This is a super common plugin that allows you to give some automatic structure to new documents. For me, a blog post has some essential stuff like a title, date, categories, but also the ability to pin a post to the front page or to keep it as a draft and not publish it. Some of these frontmatter properties are defined by **mkdocs** (more on that below).

![[attachments/Pasted image 20251120152540.png]]
///caption
A screenshot of my blog template. Pretty basic!
///

#### QuickAdd

QuickAdd is a little complicated but it's a way to create a blog post and put it in the correct folder `posts` and apply the correct template in one quick command.

#### Commander

This plugin is a way to add your own buttons onto the Obsidian interface. If you see the plus sign at the bottom of the left part of the Obsidian window: that is a button that will fire the QuickAdd command I created, thus creating a blog post in the right place. 

## mkdocs

[mkdocs](https://www.mkdocs.org/) is a static site generator that's primarily intended for people to use it to do software documentation. But it's got a blog plugin and I have found it's potentially interesting for documenting your own practice (i.e. something like a folio).

I mainly write in Python when I can and so I am quite comfortable with all of the below. None of it gave me any trouble, all is a pretty straightforward use of Python.

First, just make sure mkdocs is installed on your computer.

```
pip install mkdocs
```

These are a couple other things to customize how I use mkdocs

```
pip install mkdocs-literate-nav # to edit my nav through obsidian
pip install mkdocs-material # for the layout
pip install mkdocs-obsidian-bridge # to be able to use Obsidian as CMS
```

The each site will want its own directory:

```
mkdocs new my-project # for me my-project was mkdocs-blog
```

The most important file is the configuration file, `mkdocs.yml`. Here is mine:

```
site_name: somethingilearned.today

docs_dir: !ENV [DOCS_DIR, '/path/to/obsidian/contents/of/my/blog'] # Where is your Obsidian vault?
site_url: !ENV [SITE_URL, 'https://somethingilearned.today'] # When you eventually deploy it

# These are the files in that docs_dir that I want to make sure aren't included in the build!
exclude_docs: |
.DS_Store
README.md
LICENSE
.gitignore
.git
.github
.vscode
templates
OLD

plugins:
  - tags
  - obsidian-bridge
  - blog:
    blog_dir: . # this uses 'posts' right in the root of docs_dir
    post_readtime: false # I don't like '2 minute read'
    post_date_format: "d MMM YYYY"
  - literate-nav: # https://oprypin.github.io/mkdocs-literate-nav/index.html
    nav_file: NAVIGATION.md

markdown_extensions:
  - sane_lists
  - pymdownx.blocks.caption
  - admonition
  - footnotes
  - toc: # right sidebar
    title: Contents

theme:
  name: material
  custom_dir: overrides
  features:
    - navigation.top
    - navigation.indexes
    - navigation.sections
  palette:
	scheme: slate
	primary: white 
	accent: black
  font:
	text: Arimo # Georgia # Alegreya #Lato # Open Sans #Karla 
	code: Roboto Mono
  logo: assets/images/logo.png # Optional, see my file system
  favicon: assets/images/logo.ico # ditto

extra_css:
  - assets/stylesheets/extra.css # Optional, see my file system
```

There are a couple of places where I make custom configurations to the theme. They are the logo, a few extra styles, and also a small modification to the template.

![[attachments/Pasted image 20251120154828.png]]

## GitHub

Now I create a [Git repository for this `mkdocs-blog`](https://github.com/sdockray/mkdocs-blog) directory and I rarely edit it. I keep it separate from my [Obsidian vault, which is another repository](https://github.com/sdockray/website-blog), and which I am *always* editing.

I've got a [GitHub Action workflow](https://github.com/sdockray/website-blog/blob/main/.github/workflows/ci.yml) that runs whenever I push from my Obsidian Vault. It pulls the mkdocs configuration and then builds the site on GitHubs servers using that configuration plus my content files and publishes to a GitHub pages site, which is where it is hosted.