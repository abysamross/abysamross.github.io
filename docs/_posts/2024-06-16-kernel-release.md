---
layout: post
title: "Where does your local Linux Kernel build get its kernel release version string from?"
date: 2024-06-16 13:35:00 +0530
tags: kernel makefile build release
categories: kernel build
---
***DISCLAIMER*** This post is not intended to give an understanding of the Linux
Kernel build (*Kbuild*) mechanism or the Makefiles.\
For knowing more about the same head over to [here](https://docs.kernel.org/kbuild/makefiles.html).\
<!--exstart-->\
On building your local Linux Kernel source, the build gets a release version string which can be viewed by *making* the generic target *kernelrelease* from the source basedir:\
```$make kernelrelease```\
You can also read the same from ```include/config/kernel.release``` if this file was
created by any of the target builds like *all*, *vmlinux*, *modules*, etc.\
In this post let us take a look at how this kernel release string is formed.\
<!--exend-->
