Title: Blogging in Markdown with Obsidian, GitHub Pages, and Pelican
Date: 2025-07-02 09:01:02
Category: Musings

From 2023-2024 this blog was written primarily on [bearblog.dev](https://bearblog.dev), a free blogging platform where you write your posts in [Markdown](https://en.wikipedia.org/wiki/Markdown). I've been on a bit of a hiatus from posting for the last few months but since you're reading this obviously I'm back. 

This time I'm using some additional tools and I thought I'd share my setup. 

I write my posts in [Obsidian](obsidian.md) which does a great job of rendering Markdown while letting me edit. I can also take advantage of Obsidian's [Templates](https://help.obsidian.md/plugins/templates) core plugin to fill in [frontmatter](https://help.obsidian.md/properties) like title, date and category used by the static site generator. 

The site is served by [GitHub Pages](https://pages.github.com/) which offers static site hosting for low-traffic sites under 1 GB. Time will tell if I end up needing more space but unless I start putting images directly into the repo I should be fine for the foreseeable future. 

I use [Pelican](https://docs.getpelican.com/en/latest/index.html) to generate the html from Markdown. I chose it entirely on a whim (it happened to be the top search result for 'python static site generator') and I have no complaints so far. I'm a fan of the minimalist design aesthetic. 

I use Pelican's [reusable GitHub Action](https://docs.getpelican.com/en/latest/tips.html#publishing-to-github-pages-using-a-custom-github-actions-workflow) to build the site whenever I push a commit to GitHub. The output html isn't stored in the repo itself, so I save a few hundred kilobytes of storage and it's pretty quick to build. 

I liked the theme used by [Mark Litwintschik's blog](https://tech.marksblogg.com/) and adding it in was as easy as adding an item and a URL to the Pelican build action's config file. 

When I want to post something I go to Obsidian, open a new note, pop in the frontmatter from a template, type up my thoughts, commit and push.

~~My only problem so far is that whenever I push a commit, GitHub runs a [Jekyll](https://jekyllrb.com/) build action which inevitably fails. The workaround for this is allegedly to add a `.nojekyll` file to the repo root but so far that has been ineffective.~~ In the reusable GitHub Action I removed the `workflow-dispatch:` item under `on:` and in GitHub I set `Code and Automation -> Pages -> Build and deployment -> Source to GitHub Actions` and was able to bypass the Jekyll `pages-build-deployment` action.