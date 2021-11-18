---
title: 'Tweaking your R console'
date: 2018-05-24
permalink: /posts/tweaking-your-r-console/
tags:
  - programming
  - R
  - Linux
---

Tweaking your system is in my opinion one of the key features of Linux. Linux offers so many features to play around with to never get bored. Lately, I discovered that there is also the possibility to tweak your R console. As I was using R heavily during my Bachelor Thesis and still use it quiet commonly, I want to present some in my opinion very useful tweaks in this post.

Firstly, all user-specific tweaking of your R console can and should be done in `~/.Rprofile`. This dotfile gets executed every time you start a new R console via your Terminal. In your `~/.Rprofile` you can specify several functions which are executed at different times. The `.First` function is called after the startup of a new R session. The `.Last` function is called when you exit a session.


### Set a startup message

It is always nice to have a startup message and as R sessions always depend on a certain working directory it is also nice to know that, even though mostly I start R via terminal were I know where I started it. Nevertheless, this information never hurts and can be easily defined as the following function in your `~/.Rprofile`:

```
.First <- function() {
  # Print a welcome message
  message("Welcome back ", Sys.getenv("USER"),"!\n","Working directory is: ", getwd())
}
```

I am also very annoyed by R's default user library destination in `~/R/` whereto I set my user library path to `~/.R/` instead, which can simply be done by using the `libPaths` function.

Finally, a little bit of color can help to more easily interpret data. Therefore I use the `colorout` package to add more visual information to the command line output.

Altogether, my `.First` function looks like this:

```
.First <- function() {
  # Print a welcome message
  message("Welcome back ", Sys.getenv("USER"),"!\n","Working directory is: ", getwd())
  # Set local lib path
  .libPaths("/home/trybnetic/.R/")

  # colorize R output,
  if (interactive()) {
    library(colorout)
  }
}
```

### Customize prompting

I often use R on different machines so it is useful for me to customize my prompt to a more "bash-like" view. By adding

```
# customize your R prompt
options(prompt = paste(paste(Sys.info()[c("user", "nodename")], collapse = "@"),"[R] > "))
```

my prompt looks more like

```
trybnetic@mars [R] > x <- rnorm(20)
```

instead of

```
> x <- rnorm(20)
```

which has the large advantage that I now easily see on which machine I am currently working.

### Setting defaults

Finally, I am quiet lazy and hate it to always choose a mirror whenever I install a package. With

```
# Set CRAN repository
options(repos = structure(c(CRAN = "https://cran.hafro.is/")))
```

I set my CRAN repository permanently. Which may be disadvantageously as there might always be mirrors with faster connection times. However, in my experience these differences do not matter much.

### Full `~/.Rprofile` file

```
.First <- function() {
  # Print a welcome message
  message("Welcome back ", Sys.getenv("USER"),"!\n","Working directory is: ", getwd())
  # Set local lib path
  .libPaths("/home/trybnetic/.R/")

  # colorize R output,
  if (interactive()) {
    library(colorout)
  }
}

# Set CRAN repository
options(repos = structure(c(CRAN = "https://cran.hafro.is/")))

# customize your R prompt
options(prompt = paste(paste(Sys.info()[c("user", "nodename")], collapse = "@"),"[R] > "))
```
