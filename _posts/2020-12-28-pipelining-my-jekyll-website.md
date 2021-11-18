---
title:  "Pipelining my jekyll website"
date:   2020-12-28
permalink: /posts/pipelining-my-jekyll-website/
tags:
  - webdev
  - programming
  - continous integration
---

As you have probably already have seen, I am using the static website
generator [jekyll](https://jekyllrb.com/) to manage and generate my
website. Jekyll is a pretty handy tool create websites even though I
have to confess that particularly in the beginning I had some problems
with getting started and get to know the mechanisms behind it. But even
after gaining some knowledge on what is going on, Jekyll itself lacks
some important mechanisms of maintenance.

In a dynamic web URLs often change, but the linking in the own website does
not. As I personally don't think that waiting for user requests on dead links
is an adequate strategy now anymore, other ways of adapting and checking my
website became more appealing to me. This guide, therefore, aims to describe
my way of solving this issue, but does not claim that it is the best.
However, to me it seems practical and easy to deploy as I knew all the
mechanisms already from [earlier projects](/projects/).

So for me the choice fall to [travis CI](https://travis-ci.com). Travis is a
Continuous Integration system that enables automatic building of code on a
cloud. I first got in touch with it while writing [pyndl](https://github.com/quantling/pyndl/). In this project we used Travis CI to automatically build
the package and run the tests we wrote for each commit we made. Running the
code we wrote on our PCs on a cloud enabled us to test the code on a variety
of different operating systems and python versions.

Testing for different OS and versions did obviously not apply for this
project. But in general testing whether the project builds and especially
also to check whether it builds over time seemed a good choice. Moreover,
the produced html output can easily be checked by a htmlproofer for broken
links or other inconsistencies. Using [html-proofer](https://github.com/gjtorikian/html-proofer) was already something that I incorporated in my
workflow locally before combing it with travis, but running it on a regular
basis seems even more appealing.

The first step to get a continuous testing of my website to run was to
register myself and the repository on [travis-ci.com](https://travis-ci.com).
The registration process is pretty straight forward and you Github Account
can be used as an identity provider which is pretty handy. After registering
you project the next big step is to add a config file for your CI in your
repository. Travis' config is written in the YML format in a .travis.yml file.

As travis offers a variety of options to run and test you code, you have to
select your configuration:

```
os: linux
dist: xenial
language: ruby
rvm:
  - 2.7.0
```

For me, I decided to choose Linux as my operating system with Ubuntu Xenial Xerus as my distribution. To spare time, only a small subset of programs is
included in the build which requires to specify the programming language and
version by yourself.

In the next step you have to decide what code you want to run to test
whatever you like. For me two things were mainly important:

1. Test whether the website is building at all
2. Test for broken links and other mistakes in the html code

To do that, every time travis is triggered it builds the site using jekyll
and runs html-proofer on the result:

```
script:
  - set -e # halt script on error
  - bundle exec jekyll build
  - bundle exec htmlproofer --http-status-ignore "999" ./_site
```

**Note:** the `--http-status-ignore "999"` helps to ignore the 999 errors produced by linkedin. {: .notice}

Finally, there are some helpful tricks you can find in the travis
documentation to speed up your installation and building routine:

```
env:
  global:
  - NOKOGIRI_USE_SYSTEM_LIBRARIES=true # speeds up installation of html-proofer
```

For me, that was the above mentioned code.

So putting all the pieces together, we obtain my [.travis.yml config file](https://github.com/Trybnetic/trybnetic.github.io/blob/master/.travis.yml) that automatically builds and tests my code:

```
os: linux
dist: xenial
language: ruby
rvm:
- 2.7.0

script:
  - set -e # halt script on error
  - bundle exec jekyll build
  - bundle exec htmlproofer --http-status-ignore "999" ./_site

env:
  global:
  - NOKOGIRI_USE_SYSTEM_LIBRARIES=true # speeds up installation of html-proofer
```

However, this code is now just triggered on commit. This obviously has
advantages such as saving computation time and also often does not make
a big difference as once the code is built it remains on your page until
a new commit is triggered and is not automatically rebuild. This, however,
is a great design choice for testing whether you code builds, but
particularly with websites you do not solely face problems with your
website. At last as long as you have linked other websites as well, other
administrators might change there urls spontaneously and without notice
breaking your links. To solve this problem, I use a cron job on travis to
automatically trigger a build once a day.

To do so, choose `More options` on your travis' project page and go to
`Settings`. Scroll down to `Cron Jobs` and add a daily, weekly or monthly
cron job on your main branch. If you deploy your website on your master
branch, choose master. If you deploy it on a gh-pages branch, choose this
one. From now on a build will be triggered on the preferred regular basis
and as long as you have enabled mail notifications, you will receive a
mail telling you that some links or other things broke since the last build.
