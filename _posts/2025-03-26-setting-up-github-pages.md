---
layout: post
title: Setting Up Github Pages
date: 2025-03-26 15:39 -0600
categories: [Tutorials, GitHub]
tags: [git]
---

## It's Time To D-D-D-D-Document!

One of the most common pieces of advice I've heard when it comes to improving your ability to manage computers - as well as a boon to your resume - is to host your own 
github page, where you can document your work to reference later, as well as show off to prospective employers. This, for me, feels long overdue as I've configured various 
linux servers and built many a project without creating any documentation on the process; that's the sort of thing you do at work, not at home, I thought. As I progress into 
more and more technical topics, I find having a repository that I can reference is a grand idea; too many of my old projects are becoming mysteries to me, and knowing 
why I did what I did will save me a lot of time when trying to revisit them years later (looking at you, Plex server).

## Setting Up The Environment

### Picking A Theme

To kick matters off, I decided to find a compelling template for my documentation. A tawdry bundle of plaintext would be murder on the eyes and present no challenge; much better 
to needlessly complicate the matter by adding flair and style with little purpose and conflate the time budget. After all, that's management at its finest, right? Pursuing 
this thought, I stumbled upon a theme that uses [Jekyll](https://jekyllrb.com/), a tool set that converts plaintext into static websites. Seeing as Github Pages apparently 
uses it, I figured it would be a good place to start.

![Jekyll Powers GitHub Pages](/assets/img/posts/jekyll-powers-github-pages.png){: w="1113" h="315" }

The theme I found (and may or may not be currently using) is called [Chirpy](https://chirpy.cotes.page/posts/getting-started/). I enjoyed the simple but fluid design, and 
I especially liked that it had a native Dark Mode; I have long since been a convert of white-text-on-black-backgrounds and was happy to see I wouldn't have to create this 
on my own, nor would I have to force those yet to embrace its beauty by allowing a toggle to Light Mode. Share your preference but accommodate what you can, I say. The process 
is well documented in the Getting Started section for Chirpy so I wont replicate its steps here, rather I will only mention the issues I encountered while following their steps.

### Correcting Permissions

Creating the initial repo by copying the starter template was easy enough. The first problem came when I went to setup the environment. I used my WSL2 install of Kali Linux 
(I probably should have installed Ubuntu or something for this, but I went with what I had, and I like Kali better anyways) and got Ruby installed. However, I ran into issues 
when trying to use gem to install Jekyll itself; namely, write-permission errors. Turns out, I hadn't specified a proper gem install directory and thus Gem was trying to install 
as if it were root. A quick search provided me with [the solution](https://github.com/orgs/rubygems/discussions/6760#discussioncomment-6514401) and how to run Gem for only the 
local account.

> I later discovered that this information is also found on the Jekyll site; whether this is new or I'm just blind, I don't know. Regardless, the process was the same, so I'll put 
the terminal commands here for future reference.
{: .prompt-info }

```terminal
echo '# Install Ruby Gems to ~/gems' >> ~/.bashrc
echo 'export GEM_HOME="$HOME/gems"' >> ~/.bashrc
echo 'export PATH="$HOME/gems/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

### Fixing Dependencies

After resolving the issue with gem and Jekyll, I tried installing Bundle, a necessary component to use Jekyll. This ended up giving me an issue with dependencies that was rather 
interesting. Attempting to execute `bundle` gave the following error:

```terminal
Fetching gem metadata from https://rubygems.org/...........
Resolving dependencies...
Fetching ffi 1.17.0 (x86_64-linux)

Retrying download gem from https://rubygems.org/ due to error (2/4): Gem::RemoteFetcher::FetchError bad response Forbidden 403 (https://rubygems.org/gems/ffi-1.17.0-x86_64-linux.gem)

Retrying download gem from https://rubygems.org/ due to error (3/4): Gem::RemoteFetcher::FetchError bad response Forbidden 403 (https://rubygems.org/gems/ffi-1.17.0-x86_64-linux.gem)

Retrying download gem from https://rubygems.org/ due to error (4/4): Gem::RemoteFetcher::FetchError bad response Forbidden 403 (https://rubygems.org/gems/ffi-1.17.0-x86_64-linux.gem)

Bundler::HTTPError: Could not download gem from https://rubygems.org/ due to underlying error <bad response Forbidden 403
(https://rubygems.org/gems/ffi-1.17.0-x86_64-linux.gem)>
  /usr/share/rubygems-integration/all/gems/bundler-2.4.20/lib/bundler/rubygems_integration.rb:497:in `rescue in
download_gem'
  /usr/share/rubygems-integration/all/gems/bundler-2.4.20/lib/bundler/rubygems_integration.rb:469:in `download_gem'
  /usr/share/rubygems-integration/all/gems/bundler-2.4.20/lib/bundler/source/rubygems.rb:481:in `download_gem'
  /usr/share/rubygems-integration/all/gems/bundler-2.4.20/lib/bundler/source/rubygems.rb:443:in `fetch_gem'
  /usr/share/rubygems-integration/all/gems/bundler-2.4.20/lib/bundler/source/rubygems.rb:427:in `fetch_gem_if_possible'
  /usr/share/rubygems-integration/all/gems/bundler-2.4.20/lib/bundler/source/rubygems.rb:161:in `install'
  /usr/share/rubygems-integration/all/gems/bundler-2.4.20/lib/bundler/installer/gem_installer.rb:54:in `install'
  /usr/share/rubygems-integration/all/gems/bundler-2.4.20/lib/bundler/installer/gem_installer.rb:16:in `install_from_spec'
  /usr/share/rubygems-integration/all/gems/bundler-2.4.20/lib/bundler/installer/parallel_installer.rb:156:in `do_install'
/usr/share/rubygems-integration/all/gems/bundler-2.4.20/lib/bundler/installer/parallel_installer.rb:147:in `block in
worker_pool'
  /usr/share/rubygems-integration/all/gems/bundler-2.4.20/lib/bundler/worker.rb:62:in `apply_func'
  /usr/share/rubygems-integration/all/gems/bundler-2.4.20/lib/bundler/worker.rb:57:in `block in process_queue'
  /usr/share/rubygems-integration/all/gems/bundler-2.4.20/lib/bundler/worker.rb:54:in `loop'
  /usr/share/rubygems-integration/all/gems/bundler-2.4.20/lib/bundler/worker.rb:54:in `process_queue'
  /usr/share/rubygems-integration/all/gems/bundler-2.4.20/lib/bundler/worker.rb:90:in `block (2 levels) in create_threads'

An error occurred while installing ffi (1.17.0), and Bundler cannot continue.

In Gemfile:
  html-proofer was resolved to 5.0.9, which depends on
    typhoeus was resolved to 1.4.1, which depends on
      ethon was resolved to 0.16.0, which depends on
        ffi
```

Turns out, this was a known issue; the `ffi` package had `-gnu` appended to the platform name for Linux. However, as 
[this post](https://github.com/ffi/ffi/issues/1105#issuecomment-2166803864) explains, `bundle` had normalized the platform as simply `linux`, causing `ffi` to search for 
`linux.gem` instead of `linux-gnu.gem`. By forcing Bundle to use the pure ruby variant, the issue was resolved:

```terminal
gem "ffi", force_ruby_platform: true
```

### Converting Git To SSH

With Jekyll and build both installed, it was time to get git going. I had my git repo cloned to my computer due to prior projects but it turned out it was using https. 
Since I had enabled 2FA on my git account since cloning it, I would need to setup a personal acess token if I wanted to continue with https. Instead, I thought it a 
good idea to convert the connection to ssh. The process was easy enough; first, I had to change the upstream to use ssh for the remote origin:

```terminal
git remote set-url origin git@github.com:collingsjj/collingsjj.github.io.git
```

I then [created a new ssh key pair](https://docs.github.com/en/get-started/getting-started-with-git/managing-remote-repositories) to use specifically for this account:

```terminal
ssh-keygen -t ed25519 -C "collingsjj@gmail.com"
```

I then added it to the ~/.ssh/config file:

```terminal
# GitHub.com
Host github.com
  PreferredAuthentications publickey
  IdentityFile ~/.ssh/github-collingsjj
```

Done!

### Build With GitHub Actions

The final piece of the puzzle was setting up GitHub to automatically build the webpages whenever an update was pushed to the main branch. This makes use of the 
[GitHub Actions](https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site#publishing-with-a-custom-github-actions-workflow) 
feature for Pages. It was a simple matter of setting the **Build and deployment** setting to use GitHub Actions as the source:

![Light mode only](/assets/img/posts/pages-source-light.png){: w="500" h="187" .light }
![Dark mode only](/assets/img/posts/pages-source-dark.png){: w="500" h="187" .dark }

With the Actions set, I pushed an update to test the setup... and it failed:

```terminal
 did not find expected '-' indicator while parsing a block collection
```

Turns out the LinkedIn section in the template had too many spaces at the start (oops). While fixing the code there I took the opportunity to create the `img` directory under assets. With 
that created, I added my `avatar.png` then updated `_config.yml`. With these changes committed, I gave it a push and viola! Build was successful; it was good to go.

## Streamlining Future Posts

With the project setup and the builds working properly, I decided to dig around and learn more about Jekyll; most tools have a lot of fun and useful features just below their surface. 
I came across a plugin that would help automate the process of creating new posts and publishing drafts. See, Jekyll uses a [Front Matter](https://jekyllrb.com/docs/front-matter/) 
block in each post file to set the layout, tags, and other metadata for the file. An example would be something like this:

```terminal
---
title: TITLE
date: YYYY-MM-DD HH:MM:SS +/-TTTT
categories: [TOP_CATEGORIE, SUB_CATEGORIE]
tags: [TAG]     # TAG names should always be lowercase
---
```

This can get extremely tedious, especially when you need to put the publish date in the post's filename - a matter that can quickly become a sore spot when moving between drafts, published 
works, etc. To streamline the post creation process, I decided to add the [`jekyll-compose`](https://github.com/jekyll/jekyll-compose) plugin to the Gemfile:

```terminal
gem 'jekyll-compose', group: [:jekyll_plugins]
```

Finally, I figured I might as well create a number of aliases to make it easier to execute. I make great use of the `.bash_aliases` file; I am an alias addict, I admit:

```terminal
# Jekyll
alias jrename="bundle exec jekyll rename"
alias jpost="bundle exec jekyll post"
alias jdraft="bundle exec jekyll draft"
alias jpublish="bundle exec jekyll publish"
```

## Wrap-Up

The project was a lot of fun. I hadn't done much with my own git repos; usually I only clone what others make and then learn their ins and outs. While I did make use of a template, it was 
still to create my own content that I can put my name to. It's a small start, but an exciting start nonetheless. The process was fairly straightforward and most of the bumps were due to 
minor oversights. I feel keeping better records of future projects will lead to better implementation and more documentation; making sure to share them here will help me internalize it all 
and apply the lessons to my own creations.

Here's to many more posts to come!