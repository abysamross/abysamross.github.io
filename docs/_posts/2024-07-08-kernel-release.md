---
layout: post
title: "Where does your local Linux Kernel build get its kernel release version string from?"
date: 2024-07-08 03:05:00 +0530
tags: linux kernel makefile kernelrelease localversion kconfig build
categories: kernel build
---
***DISCLAIMER*** This post is not intended to give an understanding of the Linux
Kernel build <span class="ckw">(Kbuild)</span> mechanism or <span class="ckw">Makefile</span>.\
For knowing more about the same head over to [here](https://docs.kernel.org/kbuild/makefiles.html).<br>
<!--exstart--><br>
On building your local Linux Kernel source, the build gets a release version string which can be viewed by <span class="gkw">making</span> the generic target <span class="ckw">kernelrelease</span> from the source base directory:
<span class="termsnip">$ make kernelrelease</span>
You can also read the same from <span class="codepath">include/config/kernel.release</span> if this file was created by any of the target builds like <span class="ckw">all, vmlinux, modules</span>, etc.<br>
In this post let us take a look at how this kernel release version string is formed.<br>
<!--exend-->
<span class="ckw">kernelrelease</span> is a <span class="gkw">generic target</span> for the
<span class="gkw">Makefile</span> at the base/root of your Linux Kernel source.
<span class="termsnip">$ make help</span> from the source base directory will give you the following <span class="gkw">help</span> output about this <span class="gkw">generic target</span>:<br>
<img class="postimgs" src="{{ '/assets/images/makehelpout.png' }}" alt='make help output'><br>

<h3 id="configurator">configurators</h3>
You need to <span class="gkw">make</span> any one of the <span class="gkw">configurators</span> <span class="ckw">(\*config configuration targets)</span> like <span class="termsnip">$ make {old[def]|...}config</span> before <span class="ckw">$ make -s kernelrelease</span> can work.
 These <span class="gkw">configurators</span> make sure that a <span class="codepath">./.config</span> file is written to the source base directory.
 The <span class="codepath">.config</span> file contains all the <span class="ckw">CONFIG_\*</span> options that were selected and the unselected ones as comments.
 In addition to the <span class="codepath">.config</span> file the <span class="gkw">configurators</span> also create <span class="codepath">include/config/auto.conf</span> and <span class="codepath">include/generated/autoconf.h</span> files.
 The former contains just the <span class="ckw">CONFIG</span> options you have selected to be <span class="gkw">built-in</span> <span class="ckw">(obj-y)</span> or made a <span class="gkw">loadable module</span> <span class="ckw">(obj-m)</span> and the <span class="ckw">CONFIG</span> options with values.
 The latter contains these <span class="ckw">CONFIG</span> options converted into <span class="ckw">#define</span> directives.
The <span class="codepath">include/config/auto.conf</span> file is mandatory to execute <span class="ckw">$ make kernelrelease</span>.

Now we can peek into <span class="codepath">./Makefile</span>, the <span class="ckw">Makefile</span> at the base of the source.

<br>
<div class="blink">(ALL CODE REFERENCES ARE FROM THE <a id="bl" href="https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git/tree/?h=v6.7">v6.7</a> TAG IN THE <a id="bl" href="https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git/log/?h=linux-6.7.y">linux-6.7.y</a> BRANCH OF THE <a id="bl" href="https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git">Linux kernel stable tree.</a>)</div>
<br>

<h3>The Makefile</h3>
This <span classr="gkw">Makefile</span> sets and <span class="ckw">exports</span> many variables, defines some basic <span class="gkw">make rules</span> starting from [line 2](https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git/tree/Makefile?h=v6.7#n2) to [line 208](https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git/tree/Makefile?h=v6.7#n208). 
 These variables/rules include the default target <span class="ckw">__all</span>, the absolute path for the source tree <span class="ckw">abs_srctree</span>, the absolute path of the output directory <span class="ckw">abs_objtree</span>, <span class="gkw">make flags</span>, the make/build verbosity <span class="ckw">KBUILD_VERBOSE</span>, the <span class="ckw">make options</span>, the external modules to build <span class="ckw">KBUILD_EXTMOD</span> etc.

<img class="postimgs" id="beginning" src="{{ '/assets/images/beginning.png' }}" alt='beginning'>
{% comment %}
<img class="postimgs" src="{{ '/assets/images/verbosity.png' }}" alt='verbosity'>
<img class="postimgs" src="{{ '/assets/images/kbuild_extmod.png' }}" alt='external modules'>
{% endcomment %}
<img class="postimgs" src="{{ '/assets/images/outputdir.png' }}" alt='output directory'><br>
This <span class="ckw">Makefile</span> doesn't prefer to show <span class="termsnip">Entering directory ...</span> text in its output (i.e. prefers to be invoked with the <span class="ckw">--no-print-directory</span> option) except in the case where the <span class="gkw">make</span> invocation directory (<span class="ckw">PWD</span>) and the output directory (<span class="ckw">abs_objtree</span>) are different.
 This preference will result in an immediate recursive call with the <span class="ckw">--no-print-directory</span> option set after the user issues the first <span class="gkw">make</span> at the cmd line and this is achieved by the <span class="ckw">__sub-make</span> rule/target.
 And if the <span class="gkw">make</span> invocation directory (<span class="ckw">PWD</span>) and the output directory (<span class="ckw">abs_objtree</span>) are not the same we can end up recursing twice into the <span class="ckw">__sub-make</span> rule/target.
 (once with <span class="ckw">--no-print-directory</span> NOT set to show the <span class="gkw">"Entering directory ..."</span> message and then with <span class="ckw">--no-print-directory</span> set to start the <span class="ckw">make</span> process and avoid printing <span class="gkw">"Entering directory ..."</span> each time <span class="ckw">make</span> recurses into the <span class="gkw">Makefile</span> in a subdirectory).
 The <span class="ckw">__sub-make</span> rule is controlled by the <span class="ckw">need-sub-make</span> variable/flag.<br>
<img class="postimgs" id="sub_make" src="{{ '/assets/images/sub_make.png' }}" alt='__sub-make'><br>
The <span class="ckw">__sub-make</span> rule puts the variable definitions from [line 2](https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git/tree/Makefile?h=v6.7#n2) to [line 208](https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git/tree/Makefile?h=v6.7#n208) at the risk of being executed more than once.
 This is avoided by defining and exporting <span class="ckw">[sub_make_done](#submake)</span> at the end of first invocation of the <span class="gkw">make</span>. 
 The value of <span class="ckw">sub_make_done</span> variable is checked at [line 45](https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git/tree/Makefile?h=v6.7#n45), [here](#beginning), before the variable definition are done.

<h3>kernelrelease rule</h3>
<img class="postimgs" src="{{ '/assets/images/kernelreleaserule.png' }}" alt='kernelrelease rule'><br>
It is this <span class="ckw">make rule</span> with the target <span class="ckw">kernelrelease</span>, that gets assigned to the variable <span class="ckw">PHONY</span>, which eventually is declared as a <span class="ckw">PHONY target</span> that is responsible for displaying the kernel release version string.
 The <span class="ckw">filechk_kernel.release</span> variable in the recipe is set at 2 different places higher up in the <span class="gkw">Makefile</span>.

<h3>filechk_kernel.release variable</h3>
The first location, from the bottom-up, where <span class="ckw">filechk_kernel.release</span> is defined is under the branch for building a specific external module.
 (checkout ['Building External Modules'](https://docs.kernel.org/kbuild/modules.html) and [line 139](https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git/tree/Makefile?h=v6.7#n139) - [line 156](https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git/tree/Makefile?h=v6.7#n156) of the <span class="gkw">Makefile</span>)

<img class="postimgs" src="{{ '/assets/images/extmod_filechk_kernel.png' }}" alt='KBUILD_EXTMOD'><br>

The second is within the branch where <span class="gkw">make</span> was not
invoked for a specific external module: 
<img class="postimgs" src="{{ '/assets/images/no_extmod.png' }}" alt='!KBUILD_EXTMOD'><br>
<img id="filechk" class="postimgs" src="{{ '/assets/images/filechk_kernel.png' }}" alt='filechk_kernel.release'><br>
So it is the latter definition of <span class="ckw">filechk_kernel.release</span> that is in effect while making the generic target <span class="ckw">kernelrelease</span>.

<h3>origin function</h3>
Here you can see that <span class="ckw">ifeq ($(origin KERNELRELEASE), file)</span> dictates how <span class="ckw">filechk_kernel.release</span> is set.
 [origin](https://www.gnu.org/software/make/manual/html_node/Origin-Function.html) is a [GNU make](https://www.gnu.org/software/make/manual/html_node/index.html) text processing function that tells you where its argument text/variable came from.
 <span class="gkw">origin</span> returns a string <span class="gkw">'file'</span> if the argument text/variable was defined in a makefile.

There is also a rule for creating a file <span class="codepath">include/config/kernel.release</span> in the following lines.
 It has an interesting prerequiste <span class="ckw">FORCE</span> and a call to function <span class="ckw">filechk</span> with arguments <span class="ckw">kernel.release</span>.
 Let us explore these a bit.

<h3>FORCE prerequisite</h3>
First, let us see what this <span class="ckw">FORCE</span> prerequiste means.
 <span class="ckw">FORCE</span> prerequisite is a way of making sure that the recipe to make its target is always run whenever <span class="gkw">make</span> is run without any specific targets or when <span class="gkw">make</span> is run for that specific rule.
 This happens because <span class="ckw">FORCE</span> itself is defined way down below in the <span class="gkw">Makefile</span> as a rule without any prerequistes or recipe.
<img class="postimgs" src="{{ '/assets/images/FORCE.png' }}" alt='force'><br>
 Since the target of this rule is a non-existent file, <span class="gkw">make</span> assumes it to be updated whenever its rule is run.
 Thus all targets depending on this target (<span class="ckw">FORCE</span>) will always have their recipe run.
 It has the same effect of using <span class="ckw">.PHONY: include/config/kernel.release</span> and is useful for other versions of <span class="gkw">make</span> that do not support <span class="ckw">.PHONY</span> rules.
 The name <span class="ckw">FORCE</span> can be replaced with any other name to create this special rule. Refer [Rule without Recipes or Prerequisites](https://www.gnu.org/software/make/manual/html_node/Force-Targets.html) for more clarity.

<h3>filechk function</h3>
Now what is left of the [rule](#filechk) is the <span class="ckw">$(call filechk,kernel.release)</span> recipe.
 The function <span class="ckw">filechk</span> is defined in the file <span class="codepath">scripts/Kbuild.include</span> and is available in <span class="ckw">Makefile</span> as it is included here:
<img class="postimgs" src="{{ '/assets/images/kbuild_include.png' }}" alt='kbuild.include'><br>
This is the definition of <span class="ckw">filechk</span> function:<br>
<img class="postimgs" src="{{ '/assets/images/filechk_func.png' }}" alt='filechk
func'><br>
The function first checks if <span class="gkw">FORCE</span> is specified in the list of prerequisites, else a warning is emitted about missing <span class="gkw">FORCE</span> prerequisite and <span class="ckw">make</span> continues.
 Then it creates the directory <span class="codepath">include/config</span> if it doesn't exist.
 This is followed by setting up a <span class="ckw">trap</span> to remove/cleanup the temp file <span class="codepath">include/config/.tmp_kernel.release</span> (which we will be creating next) on script <span class="ckw">EXIT</span>.
 Then the contents of variable <span class="ckw">filechk_kernel.release</span> is written
to the temp file <span class="codepath">include/config/.tmp_kernel.release</span>.
 The script then checks if the file <span class="codepath">include/config/kernel.release</span> does not exist or if its contents does not match the contents of the temp file <span class="codepath">include/config/.tmp_kernel.release</span>.
 In either case it emits a message <span class="termsnip">UPD include/config/kernel.release</span> depending on whether <span class="gkw">make</span> verbosity was set to 1 or not and makes the temp file the new <span class="codepath">include/config/kernel.release</span> file.<br>
<span id=special>As you can see this [rule](#filechk) will be run only for <span class="termsnip">$ make include/config/kernel.release</span> invocation or for any <span class="gkw">make</span> invocation that run special rules like <span class="ckw">.PHONY</span> or <span class="ckw">FORCE</span> without recipes or prerequisites.</span>

<h3>KERNELRELEASE variable</h3>
Coming back to the [filechk_kernel.release](#filechk) rule, inorder to figure out if <span class="ckw">$(ORIGIN KERNELRELEASE)</span> evaluates to <span class="gkw">'file'</span> we need to check if <span class="ckw">KERNELRELEASE</span> was set somewhere earlier in the <span class="gkw">Makefile</span>.

<img class="postimgs" src="{{ '/assets/images/KERNELRELEASE.png' }}" alt='kernelrelease var'><br>
<span class="ckw">KERNELRELEASE</span> is read from <span class="codepath">include/config/kernel.release</span> in the branch for <span class="ckw">non mixed-build</span> targets.
But this file is not generated while making specific generic target <span class="ckw">kernelrelease</span> as explained [here](#special).
 Hence <span class="ckw">KERNELRELEASE</span> is exported with a NULL value. Though, there is nothing stopping you from creating this file before running <span class="gkw">make</span> and setting <span class="ckw">KERNELRELEASE</span> to whatever is in it during <span class="gkw">make</span>.

We now know that <span class="ckw">KERNELRELEASE</span> was exported from this very same <span class="gkw">Makefile</span> itself and that <span class="ckw">ifeq ($(origin KERNELRELEASE), file)</span> evaluates to <span class="gkw">TRUE</span>. Thus the variable [<span class="ckw">filechk_kernel.release</span>](#filechk) is set to the output of invoking the [<span class="ckw">$(srctree)/scripts/setlocalversion</span>](#filechk) script with <span class="ckw">$(srctree)</span> as the argument.

<h3>single-build & mixed-build</h3>
Before we proceed further let us digress a bit about <span
class="ckw">single-build</span> and <span class="ckw">mixed-build</span>.
<img class="postimgs" src="{{ '/assets/images/single_mixed.png' }}" alt='single-mixed-build'><br>
<span class="ckw">single-build</span> is specified only if the make goals, (<span class="ckw">MAKECMDGOALS</span>), contain at least one <span class="ckw">single-targets</span> item.
 <span class="ckw">single-targets</span> items are <span class="ckw">\*.a, \*.ko, \*.o</span> etc.

 <span class="ckw">mixed-build</span> is set if the <span class="gkw">make</span> goals contain more than one <span class="gkw">config</span> targets <span class="gkw">and/or</span> if <span class="ckw">single-targets</span> is specified along with other <span class="gkw">make</span> goals <span class="gkw">and/or</span> if the <span class="ckw">clean-targets</span> is specified along with other <span class="gkw">make</span> goals <span class="gkw">and/or</span> if the <span class="ckw">install</span> & <span class="ckw">modules_install</span> targets are specified together.

The definitions of these variables can be seen a few lines up:
<img class="postimgs" src="{{ '/assets/images/definitions.png' }}" alt='definitions'><br>

<h3>scripts/setlocalversion</h3>
<span class="codepath">scripts/setlocalversion</span> is the script that is eventually invoked to get the <span class="ckw">kernelrelease</span> string which is assigned to the <span class="ckw">filechk_kernel.release</span> variable.
 This script will try to get the values of the following variables/items if they are set/available:
<ul>
    <li id="lit"><span class="ckw">KERNELVERSION</span></li>
    <li id="lit"><span class="ckw">file_localversion</span></li>
    <li id="lit"><span class="ckw">config_localversion</span></li>
    <li id="lit"><span class="ckw">LOCALVERSION</span></li>
    <li id="lit"><span class="ckw">scm_version</span></li>
</ul>

<img class="postimgs" src="{{ '/assets/images/setlocalversion.png' }}" alt='setlocalversion'><br>

<h3>KERNELVERSION</h3>
<span class="ckw">KERNELVERSION</span> variable is set and exported in the <span class="gkw">Makefile</span> at the source base directory.
 This is how it is formed:<br>
<img class="postimgs" src="{{ '/assets/images/version.png' }}" alt='version'>
<img class="postimgs" src="{{ '/assets/images/kernelversion.png' }}" alt='kernelversion'><br>

<h3>file_localversion</h3>
<span class="ckw">file_localversion</span> is formed from the contents of localversion* files, if any, in the build and source directories.
 The <span class="ckw">collect_files</span> function is invoked to read and get the contents of all the localversion* files.<br>
<img class="postimgs" src="{{ '/assets/images/file_localversion.png' }}" alt='file_localversion'><br>

<h3>config_localversion</h3>
<span class="ckw">config_localversion</span> is set to the value provided to the <span class="ckw">CONFIG_LOCALVERSION</span> config option while running a configurator like <span class="ckw">menuconfig</span> and generating the <span class="codepath">.config</span> file.
<img class="postimgs" src="{{ '/assets/images/config_localversion.png' }}" alt='config_localversion'><br>

<h3>scm_version</h3>
This script forms the <span class="gkw">full scm version string</span> if the <span class="ckw">CONFIG_LOCALVERSION_AUTO</span> config option is set in the <span class="codepath">.config</span> file.
  If this config option is **NOT** set and <span class="ckw">LOCALVERSION</span> is also not set, then the script goes for <span class="gkw">short scm version string</span>.
<img class="postimgs" src="{{ '/assets/images/scm_version.png' }}" alt='scm_version'><br>
 Finally let us explore the <span class="ckw">scm_version</span> function:
<img class="postimgs" src="{{ '/assets/images/scm_version_func1.png' }}" alt='scm_version_func1'><br>
The function sets variables <span class="ckw">short</span> and <span class="ckw">no_dirty</span> to <span class="ckw">true</span> based on the options it was passed.
 It then does a <span class="ckw">cd</span> to the source directory the script was invoked with.
 <span class="ckw">git rev-parse --show-cdup</span> is run to make sure that we are at the source base directory i.e. to ensure that source directory passed to the script as argument is indeed the actual source base directory.
 <span class="ckw">git rev-parse --verify HEAD</span> is then run to make sure that the HEAD is pointing to a valid commit.
 The <span class="ckw">version_tag</span> variable is formed by stripping the <span class="ckw">SUBLEVEL</span> part from the <span class="ckw">KERNELVERSION</span> if it's 0 and then prefixing a 'v' to the resulting string.
<img class="postimgs" src="{{ '/assets/images/scm_version_func2.png' }}" alt='scm_version_func2'><br>
Next the <span class="ckw">tag</span> & <span class="ckw">desc</span> variables are formed.
 First <span class="ckw">tag</span> is obtained from the contents of the <span class="ckw">file_localversion</span> variable, (which was formed by reading localversion* files from the source directory and build directories), after removing any leading '-'. 
 Then the function checks if such a tag exists using <span class="ckw">git describe --match=$tag</span> and stores the result in <span class="ckw">desc</span> variable. 
 If a matching <span class="ckw">tag</span> could not be found in the previous step, the function sets <span class="ckw">tag</span> to <span class="ckw">version_tag</span> and checks again.
<img class="postimgs" src="{{ '/assets/images/scm_version_func3.png' }}" alt='scm_version_func3'><br>
If a matching <span class="ckw">tag</span> was till not found, it implies that we either do not have the <span class="ckw">tag</span> we are searching for or that we have moved past the tagged commit with later commits.
 In either case if the <span class="ckw">short</span> option was set, the function just prints (<span class="ckw">echo</span>) a "+" and returns.
 If <span class="ckw">short</span> option was not set, the function checks if it is the case that we have moved past the <span class="ckw">tag</span> by later commits and if so, the function prints the commit index (distance) of latest commit from the tagged commit as a 5 chars wide string padded with 0s to left and prefixed with a "-".
 The function follows this by printing the first 12 chars from the <span class="gkw">HEAD</span> commit (the latest commit itself) prefixed with a "-g".
 The function returns from here if the <span class="ckw">no_dirty</span> option was set.<br>
<img class="postimgs" src="{{ '/assets/images/scm_version_func4.png' }}" alt='scm_version_func4'><br>
At last, if the <span class="ckw">no_dirty</span> option was not set, the function prints "-dirty" if we have uncommitted changes in our source tree.

<h3>Illustrations</h3>
Some  kernelrelease string examples.<br><br>

With <span class="ckw">CONFIG_LOCALVERSION</span> not set and <span class="ckw">CONFIG_LOCALVERSION_AUTO</span> turned OFF/ON.<br>
<img class="termimgs" src="{{ '/assets/images/ex1.png' }}" alt='example1'><br>
With <span class="ckw">CONFIG_LOCALVERSION</span> set to <span class="ckw">"-asr"</span> and <span class="ckw">CONFIG_LOCALVERSION_AUTO</span> turned OFF/ON.<br>
<img class="termimgs" src="{{ '/assets/images/ex2.png' }}" alt='example2'><br>
With <span class="ckw">CONFIG_LOCALVERSION</span> set to <span class="ckw">"-asr"</span> and <span class="ckw">CONFIG_LOCALVERSION_AUTO</span> turned OFF and uncommitted changes.<br>
<img class="termimgs" src="{{ '/assets/images/ex3.png' }}" alt='example3'><br>
With <span class="ckw">CONFIG_LOCALVERSION</span> set to <span class="ckw">"-asr"</span> and <span class="ckw">CONFIG_LOCALVERSION_AUTO</span> turned OFF and later commits than the release tag.<br>
<img class="termimgs" src="{{ '/assets/images/ex4.png' }}" alt='example4'><br>
With <span class="ckw">CONFIG_LOCALVERSION</span> set to <span class="ckw">"-asr"</span> and <span class="ckw">CONFIG_LOCALVERSION_AUTO</span> turned OFF and later commits than the release tag and uncommitted changes (same output as above).<br>
<img class="termimgs" src="{{ '/assets/images/ex4.png' }}" alt='example4'><br>
With <span class="ckw">CONFIG_LOCALVERSION</span> set to <span class="ckw">"-asr"</span> and <span class="ckw">CONFIG_LOCALVERSION_AUTO</span> turned ON and later commits than the release tag.<br>
<img class="termimgs" src="{{ '/assets/images/ex5.png' }}" alt='example5'><br>
With <span class="ckw">CONFIG_LOCALVERSION</span> set to <span class="ckw">"-asr"</span> and <span class="ckw">CONFIG_LOCALVERSION_AUTO</span> turned ON and later commits than the release tag and uncommitted changes.<br>
<img class="termimgs" src="{{ '/assets/images/ex6.png' }}" alt='example6'><br>

<h3>The End</h3>
With this we have come to the end of this post!<br>
Hope you had a good time reading.<br><br>
-- Aby

{% comment %}
TODO:
1. show the reader how filechk_kernel.release if formed.
2. if needed introduce scripts/Kbuild.include and filechk_.
3. show the sub make in Makefile.
4. show the need-config flag being set before filechk_kernel.release is called. 
5. then show the scripts/setlocalversion file.
6. explain how the local version string gets appended on setting the config option.
7. explain dirty, git commit hash, getting appended on config auto version being set. 
{% endcomment %}
{% comment %}
code - termsnip
code path - codepath
source keywords - ckw
generic keywords - gkw
{% endcomment %}
