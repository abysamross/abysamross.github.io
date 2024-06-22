---
layout: post
title: "Where does your local Linux Kernel build get its kernel release version string from?"
date: 2024-06-16 13:35:00 +0530
tags: kernel makefile build release
categories: kernel build
---
***DISCLAIMER*** This post is not intended to give an understanding of the Linux
Kernel build <span class="ckw">(Kbuild)</span> mechanism or the Makefiles.\
For knowing more about the same head over to [here](https://docs.kernel.org/kbuild/makefiles.html).<br>
<!--exstart--><br>
On building your local Linux Kernel source, the build gets a release version string which can be viewed by <span class="gkw">making</span> the generic target <span class="ckw">kernelrelease</span> from the source basedir:
<span class="codesnip">$make kernelrelease</span>
You can also read the same from <span class="codepath">include/config/kernel.release</span> if this file was created by any of the target builds like <span class="ckw">all, vmlinux, modules</span>, etc.<br>
In this post let us take a look at how this kernel release version string is formed.<br>
<!--exend-->

{% comment %}
code - codesnip <br>
code path - codepath <br>
source keywords - italics, code <br>
generic keywords - italics
{% endcomment %}
