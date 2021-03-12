---
title: restoring recovered files identity
id: 118
date: 2010-10-16 15:08:43
updated: 2010-10-16 15:08:43
tags:
- bash
- fat32
- recovery
- unix
---

I have a couple of one terabyte external hard drives. When I first got them I formatted them with FAT32 so that I could read and write them from both Windows and OS X.

FAT32 is not a [journaling filesystem](http://en.wikipedia.org/wiki/Journaling_file_system), which means if a write operation is interrupted the drive can end up in an inconsistent state. Within the first month of using the drive, I corrupted the filesystem. I forget exactly what happened: did I knock the power cord out? forget to eject the drive?

Luckily when this happens to a FAT32 disk the files can be recovered. Unfortunately, the filenames are lost. You get a folder called `FOUND.000` with files named `FILE0000.CHK` up to, in my case, `FILE0820.CHK`. The extension on filenames is important on Windows and on OS X as well because Finder relies on it.

Unix-like systems come with a utility, called `file`, that can determine file types by examining the contents. When I first read about file it was described as checking the [first 2-bytes](http://en.wikipedia.org/wiki/File_format#Magic_number), called the magic number or magic cookie. More modern versions must check more of the file, because they offer more information than can be contained in 2-bytes.

<pre lang="bash">➜ {ninja} FOUND.000 $ file *.CHK
FILE0004.CHK: RIFF (little-endian) data, AVI, 628 x 254, 25.00 fps, video: DivX 5, audio: MPEG-1 Layer 3 (stereo, 48000 Hz)
FILE0007.CHK: RIFF (little-endian) data, AVI, 580 x 306, 23.98 fps, video: DivX 5, audio: MPEG-1 Layer 3 (stereo, 48000 Hz)
FILE0009.CHK: RIFF (little-endian) data, AVI, 576 x 320, 23.98 fps, video: XviD, audio: MPEG-1 Layer 3 (stereo, 48000 Hz)
FILE0013.CHK: RIFF (little-endian) data, AVI, 612 x 250, 23.98 fps, video: DivX 5, audio: MPEG-1 Layer 3 (stereo, 48000 Hz)
FILE0016.CHK: RIFF (little-endian) data, AVI, 592 x 320, 25.00 fps, video: XviD, audio: MPEG-1 Layer 3 (stereo, 48000 Hz)
FILE0019.CHK: RIFF (little-endian) data, AVI, 608 x 288, 25.00 fps, video: XviD, audio: MPEG-1 Layer 3 (stereo, 48000 Hz)
FILE0022.CHK: RIFF (little-endian) data, AVI, 640 x 272, 23.98 fps, video: XviD, audio: MPEG-1 Layer 3 (stereo, 44100 Hz)
FILE0026.CHK: PNG image, 640 x 272, 8-bit/color RGB, non-interlaced
FILE0027.CHK: PC bitmap, Windows 3.x format, 640 x 272 x 32
FILE0028.CHK: PNG image, 640 x 272, 8-bit/color RGB, non-interlaced</pre>

I wanted to at least check what the contents of my FOUND files were before deleting them. I needed to get the appropriate extensions back on the filenames so that finder and other tools would work with them properly. I thought I should be able to do it completely in the shell, and best of all, it's already a REPL!

First step was to sort them by type.
<pre lang="bash">➜ {ninja} FOUND.000 $ file *.CHK | grep PNG
FILE0026.CHK: PNG image, 640 x 272, 8-bit/color RGB, non-interlaced
FILE0028.CHK: PNG image, 640 x 272, 8-bit/color RGB, non-interlaced</pre>

Next I needed the filename by cutting the first 8 characters. I did not know about `cut` before but I've certainly needed to pull a section out of a line before. I expect I'll be using it again.
<pre lang="bash">➜ {ninja} FOUND.000 $ file *.CHK | grep PNG | cut -c 1-8
FILE0026
FILE0028</pre>

I needed to turn these lines in to separate `mv` commands and `xargs` does exactly that. I had not used `xargs` beyond entirely simple commands before but the `man` page was enough to get me started. Since I was about to move files without a safety net, I wanted to do a quick test first.
<pre lang="bash">➜ {ninja} FOUND.000 $ file *.CHK | grep PNG | cut -c 1-8 | xargs -I filename echo filename.CHK filename.png
FILE0026.CHK FILE0026.png
FILE0028.CHK FILE0028.png</pre>

Looked good, so it was time for the real command.
<pre lang="bash">➜ {ninja} FOUND.000 $ file *.CHK | grep PNG | cut -c 1-8 | xargs -I filename mv -v filename.CHK filename.png
FILE0026.CHK -> FILE0026.png
FILE0028.CHK -> FILE0028.png</pre>

With the extensions corrected Finder is once again previewing and opening the files correctly. It's working well for videos and images. I think multi-volume rars are going to be more of a challenge.

This was all prompted by [Usher](http://manytricks.com/usher/) being released today. I've wanted an application to manage videos on OS X for a while.
