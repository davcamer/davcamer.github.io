---
title: installing hg subversion on windows to test it
id: 36
date: 2009-07-20 00:19:28
updated: 2009-07-20 00:19:28
tags:
- Mercurial
- Subversion
- hg
- hgsubversion
- howto
- open source
- svn
---

_If you follow along, by the end of this blog post you'll be able to run the hgsubversion tests on your Windows system. Unfortunately, it probably won't work 100% correctly for you. Like the [hgsubversion wiki](http://bitbucket.org/durin42/hgsubversion/wiki/Home) says, "You should only be using this if you're ready to hack on it, and go diving into the internals of Mercurial and/or Subversion."_

I really like [Mercurial](http://mercurial.selenic.com/wiki/Mercurial). I think Subversion is starting to get a little creaky. Even worse, for whatever reason access to the [CruiseControl.net](http://confluence.public.thoughtworks.org/display/CCNET/Welcome+to+CruiseControl.NET) [subversion repository](https://ccnet.svn.sourceforge.net/svnroot/ccnet/) is so slow for me from Australia that most operations I try eventually time out. It's quite frustrating. Mercurial keeps most of the familiar Subversion operations but adds DVCS goodness like having all history in a local repository. That would nicely solve my speed problems.

I've been maintaining a [Mercurial repository of cc.net on bitbucket](http://bitbucket.org/davcamer/ccnet/overview/) for a few months now. I use the excellent [hg subversion extension](http://bitbucket.org/durin42/hgsubversion/wiki/Home), installed on my Mac OS X partition. I don't have an instance of Mercurial on my Windows partition with the hg subversion extension installed. To install on OS X I followed [this recipe](http://farmdev.com/thoughts/64/try-out-the-mercurial-subversion-extension-hg-svn-on-mac-os-x/). I have not found a similarly detailed recipe for Windows. Here's my attempt at writing such a recipe.

The outline is:

1.  install [python 2.6](http://www.python.org/download/releases/2.6.2/)
2.  [build and install](http://mercurial.selenic.com/wiki/WindowsInstall) Mercurial
3.  download [subversion binaries](http://subversion.tigris.org/servlets/ProjectDocumentList?folderID=8100)
4.  install [svn-python bindings](http://subversion.tigris.org/servlets/ProjectDocumentList?folderID=8100)
5.  install [gnu diffutils](http://gnuwin32.sourceforge.net/packages/diffutils.htm)
6.  clone [hg subversion](http://bitbucket.org/durin42/hgsubversion/wiki/Home)
7.  configure the [win32text extension](http://mercurial.selenic.com/wiki/Win32TextExtension)
8.  install [nose](http://code.google.com/p/python-nose/)
9.  run the tests

I'll go in to more detail -- the above is at least an advanced level exercise. The entire install is a command-line exercise. Because of that, paths are very important. I use a folder `c:\code` to work with sources. I'll leave that path in my examples, but remember if you would like to change it to something else that is okay. Do remember to change it everywhere though.

#### 0\. Setting up paths

During installation I isolated myself somewhat by using a batch file to reset my path. That gave me more confidence that my existing TortoiseSVN and TortoiseHg installs wouldn't interfere with the hgsubversion install. Visual Studio installs a _Visual Studio 2008 Commmand Prompt_ shortcut on to the start menu that opens a command window with specific environment variables set. I copied it to create a similar shortcut for my install work. The shortcut uses these settings:

![screen-capture.png](http://intwoplacesatonce.com/wp-content/uploads/2009/07/screen-capture1.png)

Then it's a matter of editing `set_hg_paths.bat` to set the correct `%PATH%` value. It doesn't matter if `%PATH%` includes folders that are not yet on disk, so you can set all your paths now and the appropriate pieces will be picked up when they are available. My `set_hg_paths.bat` file contains:
<pre lang="text">set PATH=c:\python26\;c:\python26\scripts\;c:\Program Files\svn-win32-1.6.3\bin;C:\Program Files\GnuWin32\bin\;c:\windows\system32;c:\windows;c:\windows\system32\wbem;</pre>

#### 1\. Install python 2.6

Mercurial is written in python, runs on it, and hg subversion is a collection of python scripts so this is absolutely necessary. I downloaded [2.6.2](http://www.python.org/download/releases/2.6.2/) from the [python home page](http://www.python.org), and installed it to the default location of `c:\python26`.

To test this step, open a command prompt using the shortcut from step 0\. If you run the commands as shown, you should get the same output: <pre lang="text">c:\code>python --version
Python 2.6.2</pre>

#### 2\. Build and install Mercurial

I have already tried out Mercurial on Windows, using two different pre-packaged Mercurial distributions: one for [command-line](http://mercurial.berkwood.com/) and one following the [TortoiseSCM](http://bitbucket.org/tortoisehg/stable/wiki/Home) [tradition](http://www.tortoisecvs.org/).

Unfortunately, the pre-built binaries do not seem to be appropriate for extensions. They package the python run-time internally. This is nice on the one hand because people can get Mercurial with a single download. It does make it difficult to experiment however because it is not clear how to add python and hg extensions to the packaged run-time. Maybe if hgsubversion runs well enough on Windows, it will be included in the packages.

To try out hgsubversion, for now, I needed to download the Mercurial source. There are rough instructions for an [install from source](http://mercurial.selenic.com/wiki/WindowsInstall) on the Mercurial wiki. Since I don't yet have a working hg in my installation environment, I downloaded the [1.3 source archive](http://mercurial.selenic.com/release/mercurial-1.3.tar.gz) from the [Mercurial site](http://mercurial.selenic.com/release/?M=D). After downloading I unzipped to `c:\code\Mercurial_src`. The [wiki](http://mercurial.selenic.com/wiki/WindowsInstall#Overview)'s Standard procedure section describes a two-step build process using build and install targets. I followed those steps.

The build step compiles some C code. The `%PATH%` variable needs a C compiler for build to work. I have Visual Studio 2008 installed and it comes with a batch file to add the tools to the `%PATH%`. I added it on to the end of my `set_hg_paths.bat`. The compiler is only needed for this step, so you may prefer to just execute the batch file once.

<pre lang="text">set PATH=c:\python26\;c:\python26\scripts\;c:\Program Files\svn-win32-1.6.3\bin;C:\Program Files\GnuWin32\bin\;c:\windows\system32;c:\windows;c:\windows\system32\wbem;

"c:\Program Files\Microsoft Visual Studio 9.0\VC\vcvarsall.bat"</pre>

With that set, I opened another command window with the new `set_hg_paths.bat` and ran the build and install steps. I've pasted the beginning and end of the input below. I also saved the [full build and install log](http://intwoplacesatonce.com/wp-content/uploads/2009/07/mercurial_build.txt "mercurial_build.txt").
<pre lang="text">c:\code>set PATH=c:\python26\;c:\python26\scripts\;c:\Program Files\svn-win32-1.6.3\bin;C:\Program Files\GnuWin32\bin\;c
:\windows\system32;c:\windows;c:\windows\system32\wbem;

c:\code>"c:\Program Files\Microsoft Visual Studio 9.0\VC\vcvarsall.bat"
Setting environment for using Microsoft Visual Studio 2008 x86 tools.

c:\code>echo %PATH%
c:\Program Files\Microsoft Visual Studio 9.0\Common7\IDE;c:\Program Files\Microsoft Visual Studio 9.0\VC\BIN;c:\Program
Files\Microsoft Visual Studio 9.0\Common7\Tools;c:\WINDOWS\Microsoft.NET\Framework\v3.5;c:\WINDOWS\Microsoft.NET\Framewo
rk\v2.0.50727;c:\Program Files\Microsoft Visual Studio 9.0\VC\VCPackages;C:\Program Files\\Microsoft SDKs\Windows\v6.0A\
bin;c:\python26\;c:\python26\scripts\;c:\Program Files\svn-win32-1.6.3\bin;C:\Program Files\GnuWin32\bin\;c:\windows\sys
tem32;c:\windows;c:\windows\system32\wbem;

c:\code>cd Mercurial_src

C:\code\Mercurial_src>python setup.py build --force
running build
running build_py
copying mercurial\ancestor.py -> build\lib.win32-2.6\mercurial
.... full log in the attached file ....
copying contrib\win32\hg.bat -> build\scripts-2.6
running build_mo
warning: build_mo: could not find msgfmt executable, no translations will be built

C:\code\Mercurial_src>python setup.py install --force --skip-build
running install
running install_lib
copying build\lib.win32-2.6\hgext\acl.py -> c:\python26\Lib\site-packages\hgext
.... full log in the attached file ....
copying i18n\zh_TW.po -> c:\python26\Lib\site-packages\mercurial\i18n
running install_egg_info
Removing c:\python26\Lib\site-packages\mercurial-1.3-py2.6.egg-info
Writing c:\python26\Lib\site-packages\mercurial-1.3-py2.6.egg-info

C:\code\Mercurial_src></pre>

To test this step, I checked the version of hg that my command-prompt was now picking up. I added a new folder to my `%PATH%` at this point, although if you followed step 0, you will already have it included.

<pre lang="text">c:\code>echo %PATH%
c:\python26\;c:\python26\scripts\;

c:\code>hg version
Mercurial Distributed SCM (version 1.3)

Copyright (C) 2005-2009 Matt Mackall <mpm@selenic.com> and others
This is free software; see the source for copying conditions. There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.</pre>

#### 3\. Download Subversion binaries

As I said before, Mercurial is written in python. To interact with Subversion it makes calls in to Subversion binaries written in C. The easiest, cleanest way to get an up to date version of the Subversion binaries is to download and install a binary distribution of Subversion. This needs to be compatible with the binaries in the next step, so I [got both](http://subversion.tigris.org/servlets/ProjectDocumentList?folderID=8100&expandFolder=8100&folderID=8100) from Subversion's [download page](http://subversion.tigris.org/getting.html#windows) at tigris.org. I downloaded the [1.6.3](http://subversion.tigris.org/svn_1.6_releasenotes.html) binaries compiled against Apache 2.2\. After downloading the zip package, I unzipped it to `c:\Program Files\svn-win32-1.6.3`.

The other binary distributions of Subversion are great, but similarly to Mercurial the basic package is preferred here because it is more compatible with extensions. [Slik](http://www.sliksvn.com/en/download) has a very easy to install, minimal package that is great for clients where command-line or scripting is used. At my current job we use [VisualSVN Server](http://www.visualsvn.com/server/) as our Subversion repository because it is easy to install and administer on Windows. Unfortunately, I don't believe they are useful to us for installing hgsubversion.

After adding `C:\Program Files\svn-win32-1.6.3\bin` to my `set_hg_path.bat`, I tested this step by opening a new prompt and checking the version of `svn` that was being picked up:
<pre lang="text">c:\code>set PATH=c:\python26\;c:\python26\scripts\;c:\Program Files\svn-win32-1.
6.3\bin;

c:\code>svn --version
svn, version 1.6.3 (r38063)
   compiled Jun 22 2009, 09:59:12

Copyright (C) 2000-2009 CollabNet.
Subversion is open source software, see http://subversion.tigris.org/
This product includes software developed by CollabNet (http://www.Collab.Net/).

The following repository access (RA) modules are available:

* ra_neon : Module for accessing a repository via WebDAV protocol using Neon.
  - handles 'http' scheme
  - handles 'https' scheme
* ra_svn : Module for accessing a repository using the svn network protocol.
  - with Cyrus SASL authentication
  - handles 'svn' scheme
* ra_local : Module for accessing a repository on local disk.
  - handles 'file' scheme
* ra_serf : Module for accessing a repository via WebDAV protocol using serf.
  - handles 'http' scheme
  - handles 'https' scheme</pre>

#### 4\. Install svn-python bindings

This piece is the "glue-code" that allows Mercurial, written in Python, to call the binary Subversion API, written in C. To side-step as many problems as possible these should be from the same compile that created the Subversion binaries. For that reason, I downloaded [my bindings](http://subversion.tigris.org/files/documents/15/46152/svn-python-1.6.3.win32-py2.6.exe) from [the same page](http://subversion.tigris.org/servlets/ProjectDocumentList?expandFolder=91&folderID=8100) I used above. These are the python 2.6, svn 1.6.3, apache 2.2, win32 bindings packaged as an executable installer.

After downloading, I ran the executable. The setup wizard automatically detected my Python 2.6 installation.
![screen-capture-1.png](http://intwoplacesatonce.com/wp-content/uploads/2009/07/screen-capture-11.png)

#### 5\. Install gnu diffutils

Mercurial relies on a command-line `diff` program. `diff` is ubiquitous on Unix-related systems, but not a common program on Windows. There is a port of the gnu version for windows available as a package called [DiffUtils for Windows](http://gnuwin32.sourceforge.net/packages/diffutils.htm). I downloaded the [Complete package, except sources](http://gnuwin32.sourceforge.net/downlinks/diffutils.php) and ran the wizard to install it.

![screen-capture-2.png](http://intwoplacesatonce.com/wp-content/uploads/2009/07/screen-capture-2.png)

To check it, I added `C:\Program Files\GnuWin32\bin` to my `%PATH%` and ran `diff` to make sure it could be found:
<pre lang="text">c:\code>set PATH=c:\python26\;c:\python26\scripts\;c:\Program Files\svn-win32-1.
6.3\bin;C:\Program Files\GnuWin32\bin\;

c:\code>diff --version
diff (GNU diffutils) 2.8.7
Written by Paul Eggert, Mike Haertel, David Hayes,
Richard Stallman, and Len Tower.

Copyright (C) 2004 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.</pre>

#### 6\. Clone hgsubversion

Having Mercurial working locally at this point, I used it to clone the [hgsubversion](http://bitbucket.org/durin42/hgsubversion/wiki/Home) code from [bitbucket](http://bitbucket.org/). I used the -U option to skip the local working copy checkout, because I want to configure an extension in the repository before getting the local working copy.
<pre lang="text">c:\code>set PATH=c:\python26\;c:\python26\scripts\;c:\Program Files\svn-win32-1.
6.3\bin;C:\Program Files\GnuWin32\bin\;

C:\code>hg clone -U http://bitbucket.org/durin42/hgsubversion/
destination directory: hgsubversion
requesting all changes
adding changesets
adding manifests
adding file changes
added 475 changesets with 1178 changes to 161 files</pre>

#### 7\. Configure the win32text extension

Mercurial has a convention of storing line endings for text files as line-feeds only, the Unix convention. The Windows convention is to use both carriage-return and line-feed characters to mark line endings. To work with hgsubversion source on Windows without mixing line endings I configured the [win32text extension](http://mercurial.selenic.com/wiki/Win32TextExtension) to [handle conversion](http://mercurial.selenic.com/wiki/FAQ#FAQ.2BAC8-TechnicalDetails.What_about_Windows_line_endings_vs._Unix_line_endings.3F) as files move in and out of the working copy.

To turn it on, I opened up `c:\code\hgsubversion\.hg\hgrc` and modified the content as follows:
<pre lang="ini">[paths]
default = http://bitbucket.org/durin42/hgsubversion/

[extensions]
hgext.win32text=

[encode]
# Encode files that don't contain NUL characters.
** = cleverencode:

[decode]
# Decode files that don't contain NUL characters.
** = cleverdecode:

[patch]
# Turn on special handling for line-endings at patch-time
eol = crlf

[hooks]
# Reject commits which would introduce windows-style text" files
pretxncommit.crlf = python:hgext.win32text.forbidcrlf</pre>

After changing the config, I updated my working copy. Although there is a warning, I'm not sure how to best handle it.

<pre lang="text">C:\code>cd hgsubversion

C:\code\hgsubversion>hg update
WARNING: tests/test_push_eol.py already has CRLF line endings
and does not need EOL conversion by the win32text plugin.
Before your next commit, please reconsider your encode/decode settings in
Mercurial.ini or C:\code\hgsubversion\.hg\hgrc.
125 files updated, 0 files merged, 0 files removed, 0 files unresolved</pre>

Feeling a bit curious I opened up the files in [Notepad++](http://notepad-plus.sourceforge.net/uk/site.htm), which can show line ending characters. All the files now had CR-LF endings, as expected.

#### 8\. Install nose

[nose](http://code.google.com/p/python-nose/) is an alternative Python test runner that hgsubversion uses. There is an explanation in the hgsubversion [`README` file](http://bitbucket.org/durin42/hgsubversion/src/tip/README). Since I thought there would probably be test failures, I decided to install nose.

The easiest way to install nose is with a script called `easy_install`. The Windows Python distribution does not come with the setuptools package that provides `easy_install` by default, so I installed it first. There is not yet a self-installing package of setuptools for Windows and Python 2.6\. This [stack overflow question](http://stackoverflow.com/questions/309412/how-to-setup-setuptools-for-python-2-6-on-windows) referred me to [this script](http://peak.telecommunity.com/dist/ez_setup.py) that downloaded and installed setuptools for me. I had to use _Save As..._ in the browser to avoid ending up with html tags and escapes in the script.

<pre lang="text">c:\code>set PATH=c:\python26\;c:\python26\scripts\;c:\Program Files\svn-win32-1.
6.3\bin;C:\Program Files\GnuWin32\bin\;

c:\code>cd ez_setup

C:\code\ez_setup>python ez_setup.py
Downloading http://pypi.python.org/packages/2.6/s/setuptools/setuptools-0.6c9-py
2.6.egg
Processing setuptools-0.6c9-py2.6.egg
Copying setuptools-0.6c9-py2.6.egg to c:\python26\lib\site-packages
Adding setuptools 0.6c9 to easy-install.pth file
Installing easy_install-script.py script to c:\python26\Scripts
Installing easy_install.exe script to c:\python26\Scripts
Installing easy_install-2.6-script.py script to c:\python26\Scripts
Installing easy_install-2.6.exe script to c:\python26\Scripts

Installed c:\python26\lib\site-packages\setuptools-0.6c9-py2.6.egg
Processing dependencies for setuptools==0.6c9
Finished processing dependencies for setuptools==0.6c9</pre>

With easy_install working, it **is** easy to install nose:
<pre lang="text">c:\code>set PATH=c:\python26\;c:\python26\scripts\;c:\Program Files\svn-win32-1.
6.3\bin;C:\Program Files\GnuWin32\bin\;

c:\code>easy_install nose
Searching for nose
Reading http://pypi.python.org/simple/nose/
Reading http://somethingaboutorange.com/mrl/projects/nose/
Best match: nose 0.11.1
Downloading http://somethingaboutorange.com/mrl/projects/nose/nose-0.11.1.tar.gz

Processing nose-0.11.1.tar.gz
Running nose-0.11.1\setup.py -q bdist_egg --dist-dir c:\docume~1\admini~1\locals
~1\temp\easy_install-uirbjm\nose-0.11.1\egg-dist-tmp-kslr2c
no previously-included directories found matching 'doc\.build'
Adding nose 0.11.1 to easy-install.pth file
Installing nosetests-2.6-script.py script to c:\python26\Scripts
Installing nosetests-2.6.exe script to c:\python26\Scripts
Installing nosetests-script.py script to c:\python26\Scripts
Installing nosetests.exe script to c:\python26\Scripts

Installed c:\python26\lib\site-packages\nose-0.11.1-py2.6.egg
Processing dependencies for nose
Finished processing dependencies for nose</pre>

#### 9\. Run the tests

After all that, I ran the tests by executing `nosetests` in the hgsubversion folder. The results give many failures. I started working to correct them a couple of weeks ago and posted my work to [bitbucket](http://bitbucket.org/davcamer/hgsubversion-py26-win/). I'll have to write more about that in a subsequent post. Some of the failures today look new to me.

The start and end of the log follows. Again, I've attached the [full log](http://intwoplacesatonce.com/wp-content/uploads/2009/07/hgsubversion_tests.txt "hgsubversion_tests.txt").
<pre lang="text">c:\code>set PATH=c:\python26\;c:\python26\scripts\;c:\Program Files\svn-win32-1.6.3\bin;C:\Program Files\GnuWin32\bin\;c
:\windows\system32;c:\windows;c:\windows\system32\wbem;

c:\code>"c:\Program Files\Microsoft Visual Studio 9.0\VC\vcvarsall.bat"
Setting environment for using Microsoft Visual Studio 2008 x86 tools.

c:\code>echo %PATH%
c:\Program Files\Microsoft Visual Studio 9.0\Common7\IDE;c:\Program Files\Microsoft Visual Studio 9.0\VC\BIN;c:\Program
Files\Microsoft Visual Studio 9.0\Common7\Tools;c:\WINDOWS\Microsoft.NET\Framework\v3.5;c:\WINDOWS\Microsoft.NET\Framewo
rk\v2.0.50727;c:\Program Files\Microsoft Visual Studio 9.0\VC\VCPackages;C:\Program Files\\Microsoft SDKs\Windows\v6.0A\
bin;c:\python26\;c:\python26\scripts\;c:\Program Files\svn-win32-1.6.3\bin;C:\Program Files\GnuWin32\bin\;c:\windows\sys
tem32;c:\windows;c:\windows\system32\wbem;

C:\code>cd hgsubversion

C:\code\hgsubversion>python setup.py build
running build
running build_py
copying hgsubversion\util.py -> build\lib\hgsubversion

C:\code\hgsubversion>nosetests
..FEE..F..FFEEFFEEFFEEEEEEEEFFF..EEFFFFFEEFFFFEF.....FF..FFF.FFFFF..EE..FF.EEEEEEEEEEEEEEEEEEE..EEFF..FFFFFFFFFFFFFFFFFF
FFFF..FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFEEEEEEFFFFF...EEEEEEFFF......EEEFFEEF
======================================================================
ERROR: test_externals (tests.test_externals.TestFetchExternals)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "C:\code\hgsubversion\tests\test_externals.py", line 60, in test_externals
    self.assertEqual(ref0, repo[0]['.hgsvnexternals'].data())
  File "c:\python26\lib\site-packages\mercurial\context.py", line 84, in __getitem__
    return self.filectx(key)
  File "c:\python26\lib\site-packages\mercurial\context.py", line 159, in filectx
    fileid = self.filenode(path)
  File "c:\python26\lib\site-packages\mercurial\context.py", line 148, in filenode
    return self._fileinfo(path)[0]
  File "c:\python26\lib\site-packages\mercurial\context.py", line 143, in _fileinfo
    _('not found in manifest'))
LookupError: .hgsvnexternals@000000000000: not found in manifest
-------------------- >> begin captured stdout << ---------------------
no changes found

--------------------- >> end captured stdout << ----------------------
.... full text in attached log ....
======================================================================
FAIL: test_rebase (tests.test_utility_commands.UtilityTests)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "C:\code\hgsubversion\tests\test_utility_commands.py", line 176, in test_rebase
    self.assertEqual(self.repo['tip'].parents()[0].parents()[0], self.repo[0])
AssertionError: <changectx 000000000000> != <changectx 8bc599092f77>
-------------------- >> begin captured stdout << ---------------------
no changes found
2 files updated, 0 files merged, 0 files removed, 0 files unresolved
Nothing to rebase!

--------------------- >> end captured stdout << ----------------------

----------------------------------------------------------------------
Ran 225 tests in 161.242s

FAILED (errors=59, failures=132)</pre>
That's it for now. I believe at this point you could install the extension. Things might not work correctly though. I'm going to attempt to get more tests working locally for myself before I try turning it on and cloning anything. Even after that I'll need to try it with a few different Subversion repositories before using it for code I care about. SCM is too central to my development practices for me to use a shaky system.
