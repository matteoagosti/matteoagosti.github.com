---
layout: post
title: "Drupal website development using a GIT workflow"
date: 2013-01-17 21:10
comments: true
categories: [Development, Drupal, GIT]
---

I've been using Drupal for almost 2 years now and I really consider it to be on of the best solutions available when you need a powerful CMS. The source code of Drupal and all its modules are managed using GIT and this dramatically simplifies all code management. The goal of this article is to share a workflow entirely based on GIT that supports the creation, deployment and update of a Drupal website. I won't cover aspects of Drupal setup and management, just the file handling part and I'll assume a bit of familiarity with GIT.

<!-- more-->

## Setup

The approach I'm going to describe here will add Drupal as one of the remote origin of the project and all Drupal's module as GIT submodules. The latter has several hiccups especially when switching among branches of the main project, but being relatively careful  will keep everything safe. In case you want to know more about GIT submodules in general check out their [great guide](http://git-scm.com/book/en/Git-Tools-Submodules "Git Tools - Submodules").

The first step is to create a folder called `mywebsite` and control its source code using GIT:

{% codeblock lang:bash %}
mkdir mywebsite
cd mywebsite
git init
touch README
git add README
git commit -am "Initial commit"
{% endcodeblock %}

If you are familiar with GIT you'll notice that we created a local repository that has no remote origin, thus any committed changes will stay local. For the purpose of this example I won't add a remote repository, but you could easily do it with the following command:

{% codeblock lang:bash %}
git remote add origin git://example.com/git.git/
{% endcodeblock %}

Now let's rename the default branch `master` into `development` (where all changes will happen) and create a new one for the `production` environment:

{% codeblock lang:bash %}
git branch -m master development
git branch production
{% endcodeblock %}

Notice that I'm creating the `production` branch without the usual `-b` flag as I don't want to set my current branch to it and stick with `development`.

Now it's time to add Drupal core as one of the remote repositories so that we can pull the latest version. At the moment of writing the latest core version is 7.19, but I'm going to pull the 7.18 as I want to cover an example of core update:

{% codeblock lang:bash %}
git remote add drupal git://drupalcode.org/project/drupal.git
git pull drupal 7.18
{% endcodeblock %}

With the `pull` not only we are asking GIT to `fetch` the 7.18 tag, but also to `merge` it with our current branch (`development`). As a result, GIT will prompt you to add a comment message to the merge operation; you can leave the default message (assuming it fired up `vi` just save and quit with `:wq`).

After the core is installed we can add modules. In this example I'll simply add one, but you can repeat the command for all modules you need. Simply do not forget to add the branch of the module you want to work on and the path where it should be installed:

{% codeblock lang:bash %}
git submodule add --branch 7.x-1.x http://git.drupal.org/project/ctools.git sites/all/modules/ctools
git commit -am "Adding module ctools 7.x-1.x"
{% endcodeblock %}

## Deployment

Assuming that `production` will be our deployment branch we just need to check it out and merge changes done in `development`. Let's start by moving into `production`:

{% codeblock lang:bash %}
git checkout production
{% endcodeblock %}

You'll notice that GIT will generate an error as it won't be able to remove the `sites/all/modules/ctools` folder due to being not empty.  This is what I previously defined as the hiccup of working with GIT submodules. When you switch branch GIT actually keeps all submodules folder because it considers them as untracked changes;  a submodule, in fact, is seen as a simple reference, so when you switch to a branch that does not have this reference the corresponding folder appears with no mapping and thus is considered as untracked change. To prevent this issue, every time you switch branch into a GIT project that has submodules you have to first force cleaning the repository and then update the submodule content. The same applies when you have different versions of the same submodule across your branches. In order to prevent issues, when you switch branch you should run the following commands:

{% codeblock lang:bash %}
git clean -ffd
git submodule update --init --recursive
{% endcodeblock %}

Just be aware that by doing this you are going to **loose untracked changes** so before switching branch be sure to commit! As the `production` environment is empty at this stage, the `submodule` command won't do anything.

Now that we are on the `production` branch we can merge changes done in the `development` one:

{% codeblock lang:bash %}
git merge development
{% endcodeblock %}

After the merge we now have to update all submodules, as they do not get automatically updated:

{% codeblock lang:bash %}
git submodule update --init --recursive
{% endcodeblock %}

## Handling updates

Now let's try to update Drupal core. Before doing that we are going to switch to the `development` branch:

{% codeblock lang:bash %}
git checkout development
git clean -ffd
git submodule update --init --recursive
{% endcodeblock %}

Notice that after checking out I always clean for untracked changes and update submodules as to prevent the issue mentioned before. In a real scenario you won't probably need to switch branches continuously as you are going to have the development environment on a machine and the production onto another.

Since we want to pull the last version of Drupal core (7.19 at the moment of writing) we first need to `fetch` latest code and then `merge` the latest branch. The `fetch` operation could be omitted now since Drupal code  probably won't change in the middle of this example, but I included it for reference:

{% codeblock lang:bash %}
git fetch drupal
git merge 7.19
{% endcodeblock %}

Before checking out the latest branch we had to fetch all updates as to have all changes (obviously this is included as reference, since for this example the code ). Assuming you tested out that everything works fine you can deploy by first checking out the `production` branch:

{% codeblock lang:bash %}
git checkout production
git clean -ffd
git submodule update --init --recursive
{% endcodeblock %}

and then merging changes and updating all submodules:

{% codeblock lang:bash %}
git merge development
git submodule update --init --recursive
{% endcodeblock %}

What if you need to update a module? Let's switch back to the development branch for an example:

{% codeblock lang:bash %}
git checkout development
git clean -ffd
git submodule update --init --recursive
{% endcodeblock %}

Whenever you want to update a GIT submodule you need to go in its directory as it is a standalone GIT repository itself, `fetch` the latest source code and `checkout` the branch you need:

{% codeblock lang:bash %}
cd sites/all/modules/ctools
git fetch
git checkout 7.x-1.2
{% endcodeblock %}

You my ask why `checkout` and not `merge` as we did with the core. Well, the `checkout` operation applied to the core would overwrite all GIT project settings resulting in a start from scratch. For the module we do not need to track change so the `checkout` is more appropriate. In case you decide to work on the module code, than you need to track changes, work on your branch and then do the `merge`. However, the latter is a real complex scenario that deserves an article on its own.

Now **go back to the root** of our GIT project, the `mywebsite` folder, and commit your changes:

{% codeblock lang:bash %}
git commit -am "Updating module ctools to 7.x-1.2"
{% endcodeblock %}

It is important to execute the above command from the root of the project as we commit on the GIT project not on the GIT submodule.

To deploy changes to the `production` environment the procedure is the same as mentioned before, first we `checkout` the `production` branch:

{% codeblock lang:bash %}
git checkout production
git clean -ffd
git submodule update --init --recursive
git merge development
{% endcodeblock %}

and then we `merge` changes and update submodules:

{% codeblock lang:bash %}
git merge development
git submodule update --init --recursive
{% endcodeblock %}

## Conclusions

With this article I wanted to describe a full GIT workflow to support the creation, deployment and update of websites based on Drupal, with modules handling throughout GIT submodules. There are a couple of drawbacks with this approach mainly due to branch switching when using submodules, however many are the benefits when all your code is always synced up and easy to update. If you want to dig more into details I highly recommend reading the [GIT guide](http://drupal.org/node/803746 "Building a Drupal site with Git") on Drupal website.

