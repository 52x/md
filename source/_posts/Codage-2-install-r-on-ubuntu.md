title: Install R and RStudio on Ubuntu
date: 2015-09-20 19:18:43
categories: Codage|编程
tags: [Ubuntu, R, Linux]
---
I wanted to try Linux system thus I formatted an old laptop bought 7 years ago (HP Pavilion dm3, French keyboard).
No doubt that I need R and RStudio to be installed on this laptop...
<!-- more -->

## Install R
### Software Center
In the software center, you can easily find an official R app under Category "Science & Engineering :: Mathematics".
Installed that, I tried to installed packages "dplyr" but failed - error message told me that the current version is ver. 3.0.x...
That definitely doesn't meet my needs because I'm using ver. 3.2.2 on WIN7. We need to remove it and reinstall it by cmd line.

### Cmd Lines
On Cran-R homepage, there is an index led to [Ubuntu installation instruction](https://cran.r-project.org/).
The first thing to do is to "add an entry in your /etc/apt/sources.list file" (one of the 3 lines below, anyone is OK):
``` bash
deb https://<my.favorite.cran.mirror>/bin/linux/ubuntu vivid/
deb https://<my.favorite.cran.mirror>/bin/linux/ubuntu trusty/
deb https://<my.favorite.cran.mirror>/bin/linux/ubuntu precise/
```
Where "https://<my.favorite.cran.mirror>" can be replaced by your favorate [CRAN mirror address](https://cran.r-project.org/mirrors.html)
For instance, I prefer the mirror from RStudio, so - **to who never used Linux before** - fill the line below in tab "Other Software" of the file "sources.list" under "/etc/apt/":
``` bash
deb http://cran.rstudio.com/bin/linux/ubuntu precise/
```
Again! **IT IS NOT A COMMAND!**

Remember that I've already installed R of an older version, I need to remove that beforehand. Ignore that if you never install it.
``` bash
sudo apt-get remove r-base-core
```
According to the official instruction, to install the complete R system, use:
``` bash
sudo apt-get update
sudo apt-get install r-base
```
But that didn't work, I got an error message:
``` bash
The following packages have unmet dependencies:
 r-base : Depends: r-base-core (>= 3.2.2-1ubuntu0) but it is not going to be installed
          Depends: r-recommended (= 3.2.2-1ubuntu0) but it is not going to be installed
          Recommends: r-base-html but it is not going to be installed
E: Unable to correct problems, you have held broken packages.
```

By scrolling down the instruction:
> The Ubuntu archives on CRAN are signed with the key of “Michael Rutter marutter@gmail.com” with key ID E084DAB9. 

To add the key to your system and finish the installation with the commands:
``` bash
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys E084DAB9
sudo add-apt-repository ppa:marutter/rdev
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install r-base
```

-------------------------------------------

## Install RStudio
To install RStudio, you can directly download the .deb package from [https://www.rstudio.com/products/rstudio/download/](https://www.rstudio.com/products/rstudio/download/). Once it's downloaded, software center will pop up a window asking for installation.

-------------------------------------------
Reference:
[Installing latest version of R-base](http://askubuntu.com/questions/218708/installing-latest-version-of-r-base)