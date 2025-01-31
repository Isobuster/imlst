# *.imlst and filename magic

*.imlst is a file extension, first developed for IsoBuster 5.2 and up.

<h2>Concatenate files / drives / ..</h2>

Works on real files, virtual files, drives etc.

It is a simple text file that contains a file or physical or logical drive per line.  Paths can be absolute or relative to the position of the \*.imlst file iself.
It's an alternative to providing all files/drives [on the command line](https://www.isobuster.com/help/open_spanned_files_and_drives) as it follows the same logic, so best check out the link.

This text and link serves as specification of the format. Other applications can use it as well, in the exact same way.  To improve sharing \*.imlst files between applications, please stick to the same implementation or use a different file extension if you deviate from it.

The extension of the first file in the list can be used by the application that loads it to determine what type file it is.

<h4>Example</h4> 

*concatenate two drives as one*
```
\\.\PhysicalDrive2
\\.\PhysicalDrive3
```
<h4>Example</h4> 

*treat both files as one file and treat it as a \*.dsk file, since that is the extension of the first file*
```
c:\image file 1.dsk
c:\image file 2.any // comment
```
An exception to this rule (as implemented by IsoBuster) is when the first file is an *.ibp file.  See above link for more information.
<br>A double forward slash starts a comment, either on a new line, or behind a file/drive

<h2>Virtual files</h2>

From IsoBuster 5.3 onwards there is also support for a **virtual file**.  A way to insert virtual data between, after or before file(s).
<br>The syntax is: `\\*\FileName:(-)Size:Pattern`
1. FileName can be any name and must contain at least one character.  It can have a file extension, for instance *.dsk (which would help with proper detection of the image file in case it's the first file)
2. Size must be present and is the number of bytes.  It can be decimal or hexadecimal (in which case it needs to start with 0x or end with h).  If Size is negative, for instance -512, reads in that part of the file will fail with an error.
3. Pattern is a BYTE value and is optional.  Default 0x00 is used but you can specify any value, for instance 0xFF etc.

<h4>Example</h4> 

*An iso file is missing its first 100 (2048-byte sized) blocks and hence all offsets are wrong, fix this by preceding it with a virtual file containing all zeroes*
```
\\*\.iso:2048000:0x00
c:\file.iso
```
<h2>Range in files</h2>

From IsoBuster 5.4 onwards it is also possible to select **a range inside a file, starting from a certain offset**

The syntax is somewhat confusing because it's not the actual range that is provided, rather the parts of the file that need to be ignored.
All bytes inside the Header range and all bytes inside the Footer range are ignored.  The remaining data that sits in the middle makes up for the resulting file.

The syntax is: `\\#\(Header,Footer)c:\path\filename.ext`
1. Header and Footer are in bytes.  This way it is possible to target an embedded file (inside a bigger file) or stitch together a virtual file made up from fragments of several files.

<h4>Example</h4> 

*Skip the first 512 bytes of a file and ignore the last 512 bytes of a file*
```
\\#\(512,512)c:\path\filename.ext
```
<h4>Example</h4> 

*Skip the first 512 bytes*
```
\\#\(512)c:\path\filename.ext
```
<h4>Example</h4> 

*Only ignore the last 512000 bytes of the file*
```
\\#\(,512000)c:\path\filename.ext
```
<h4>Example</h4> 

*Stitch 3 files together but skip their 512 byte headers*
```
\\#\(512)c:\path\1.ext
\\#\(512)c:\path\2.ext
\\#\(512)c:\path\3.ext
```
To target a zip file that is embedded in a different type file at a certain offset, provide a double range.
The first range works on the unzipped content, the second range tells IsoBuster where the zip file is located in the parent file.

<h4>Example</h4> 

*target a zip file that happens to sit at offset 1000 inside a different file (use virtual file to open the bin as zip)*
```
\\*\.zip:0|\\#\()\\#\(1000)c:\path\file_with_zip_inside.bin
```
<h4>Example</h4> 

*target a zip file that happens to sit at offset 1000 inside a different file (use zip prefix to open the bin as zip, see next topic)*
```
\\#\zip()\\#\()\\#\(1000)c:\path\file_with_zip_inside.bin
```
<h4>Example</h4> 

*target a zip file that happens to sit at offset 1000 inside a different file but ignore the first 20 bytes of the decompressed file*
```
\\#\zip()\\#\(20)\\#\(1000)c:\path\file_with_zip_inside.bin
```
<h2>Target file in zip file</h2>

From IsoBuster 5.5 onwards it is also possible to automatically select **a target** inside a zip file.

Wildcards are supported.  IsoBuster 5.5 onwards supports (most) image file types **including their associated files** inside a zip file.
To explain, in the case of a *.cue and the cue file references one or more *.bin or *.wav or .. files, they can all be located inside the same zip file,
and IsoBuster will find them and open them (in memory, without creating temporary files).

<h4>Example</h4>

```
\\#\zip(*.cue)c:\path\zipped_cue+iso.zip
```
Latter example can also exist in this form:
```
c:\path\zipped_cue+iso.(..cue).ibzip
```
When *.ibzip is associated with IsoBuster you can open it by simply double clicking the file.  If the filename syntax is *.(..ext).ibzip then IsoBuster will look for the first *.ext file inside the zip.
The first dot (\.) inside the brackets is replaced by the wildcard asterix (\*).  If there is no leading dot (\.) the full name must match with a file inside the zip file.

<h2>Help detect *.gz or *.bz2</h2>

From IsoBuster 5.5 onwards it is also possible to help IsoBuster detect \*.gz or \*.bz2 files that don't have the proper extension (\*.gz or \*.bz2)

If the file doesn't have an extension that helps IsoBuster detect the compression, you can use the proper prefix to let IsoBuster know

<h4>Example</h4>

```
\\#\gz()c:\path\gzip-compressed.bin
```
<h4>Example</h4>

```
\\#\bz2()c:\path\bzip-compressed.bin
```
<h2>Image file as generic image file</h2>

From IsoBuster 5.5 onwards it is also possible to use Image files in a multi-filename, to be treated like normal files.<br>
The syntax is: `\\#\img(Offset, BlockSize, Size, Options)c:\path\filename.ext`<br>
Offset, BlockSize and Size are in bytes, and they are optional.  Preferably let IsoBuster detect these values by itself.<br>
Size is determined by the Image File format but it can be overruled by this value.<br>
Currently only option `r` is supported, which means that reads from the image are then 'raw'.<br>
At first sight there doesn't seem to be much use for `\\#\img()`.  Let me try to explain with a useless example:

<h4>Example</h4>

```
\\#\img()c:\path\image.vdi
```
*This tells IsoBuster to load and interpret the \*.vdi file, next wrap it in a normal file and feed that to the generic image file interpreter.*
*The net effect will be the same as simply loading the \*.vdi directly, without the \\\\#\img() prefix*

However, imagine you have an image file per partition, very much like CloneZilla does for instance, then you need to be able to chain those together into one filename, so that IsoBuster can load the entire disk image with access to all partitions:

<h4>Example</h4>

```
\\#\img()c:\path\gpt.img
\\#\img()c:\path\sdc1.dd
\\#\img()c:\path\sdc2.partclone.ptcl.gz
\\#\img()\\#\gz()c:\path\sdc3.ntfsclone.img
```
*In above example all (expanded / interpreted) file sizes fully match up with the partition map in the MBR/GPT*
*In above example, the last file is a GZipped file, but the extension doesn't reveal it, so use prefix \\\\#\\gz() to let IsoBuster know*

<h4>Example</h4> 

*Assuming the first file (containing the partition map) is too small (as is mostly the case in CloneZilla for instance),*
*you can add a virtual file that takes up that space, so that everything aligns properly:*
```
\\#\img()c:\path\gpt.img
\\*\Filler:1031168
\\#\img()c:\path\sdc1.dd
..
```
Or pass the expected size to the first file:

<h4>Example</h4>

```
\\#\img(,,1048576)c:\path\gpt.img
\\#\img()c:\path\sdc1.dd
..
```
Or tell IsoBuster (via Size value -1) to explicitely use the MBR/GPT to determine the size:

<h4>Example</h4>

```
\\#\img(,,-1)c:\path\gpt.img
\\#\img()c:\path\sdc1.dd
..
```
<h4>Example</h4>

*In this example the expected \*.bin file is replaced by the \*.ibadr (and associated \*.ibdat) files.*<br>
*\*.cue expects raw 2352 bytes per block but normal reading from an optical disc image returns 2048 bytes per block.*
*So, force reading to be raw via option `r`*
```
tombraider2.cu2
\\#\img(,,,r)tombraider2.ibadr
```

<h2>Several parts of a (sub)file</h2>

A \*.imlst file acts as a shortcut and it is merely an easier way to provide a multi-filename to IsoBuster.
The same can still be achieved by passing the multi-filename via the command line.

A file on every line translates to one single line with those files seperated by the "|" character.

<h4>Example</h4>

```
c:\image file 1.dsk
c:\image file 2.any
```
*Translates to:*
```
c:\image file 1.dsk|c:\image file 2.any
```
*In fact, that single line in a \*.imlst file would work just the same, it just doesn't improve readability*

As long as we're using 'normal' files any such combinations can be put in the \*.imlst file

<h4>Example</h4>

```
image file 1.dsk.part01
image file 1.dsk.part02
image file 1.dsk.part03
image file 2.any
```
Which is the same as:

<h4>Example</h4>

```
image file 1.dsk.part01|image file 1.dsk.part02|image file 1.dsk.part03
image file 2.any
```
The moment we deal with virtualized files (in particular on-the-fly decompression from \*.gz, \*.zip, or an image file: \\\\#\\img()) its subfiles need to be kept together.
Assume following example (which will fail to work as expected):

<h4>Example</h4>

```
\parts\file.dsk.gz.001
\parts\file.dsk.gz.002
\parts\file.dsk.gz.003
file.dsk
```
*This will fail because the interpreter will load the first file as a gzip (and will offer transparent on the fly decompression from it) but the next file will be loaded as a regular file again, chained to the whole resulting file.*
*In fact, the first file's auto-detection would load the \*.002 and \*.003 files automatically, to complete the gzip file, and next the regular (generic image) would still have these \*.002 and \*.003 files inside it as regular files as well*
*To keep these subfiles together they need to have double "||" symbols between them.* 

Following example **will** work:

<h4>Example</h4>

```
\parts\file.dsk.gz.001||\parts\file.dsk.gz.002||\parts\file.dsk.gz.003
file.dsk
```
In case of nested virtualized files, for instance multiple gzipped fileparts that make up an image file that is part of several image files, you need to double the symbol again to: "||||"
As it is perfectly possible to pass a multi-filename to \\\\#\\img() that consists of a combination of files, some of which are zipped, others are not etc.

<h4>Example</h4>

```
\\#\img()\parts\part1.gz.001||\parts\part1.dsk.gz.002||\parts\part1.dsk.gz.003||part2.dsk
file.dsk
```
To improve readability, double "||" can be replaced by "|" again, by putting the content between "<>".  Following example is exactly the same, but it uses <>

<h4>Example</h4>

```
\\#\img()<<\parts\part1.gz.001||\parts\part1.dsk.gz.002||\parts\part1.dsk.gz.003\>|part2.dsk>
file.dsk
```
<h2>Meta files</h2>

Several types of meta files can be included<br>
These meta files contain useful information that is parsed and used, but they are not part of the bigger concatenated file.

- [imlst](https://github.com/Isobuster/imlst/blob/main/README.md#nested-imlst-files) files (see further) *(From IsoBuster 5.5 onwards)*
- [ibpt](https://github.com/Isobuster/ibpt) files *(From IsoBuster 5.5 onwards)*

<h2>Nested *.imlst files</h2>

From IsoBuster 5.5 onwards it is also possible to nest \*.imlst files<br>
Nested \*.imlst files are considered **meta** files in that they contain useful information that is parsed, but the \*.imlst files themselves are not part of the bigger concatenated file.

<h4>Example</h4>

```
\\#\img()<<\parts\part1.gz.001||\parts\part1.dsk.gz.002||\parts\part1.dsk.gz.003>|part2.dsk>
file.dsk
```
Could also be:
```
\\\\#\\img()<<\parts\gzpart.imlst>|part2.dsk>
file.dsk
```
<h4>Example</h4>

```
\clone\full-image.imlst
```
<h4>Example</h4>

```
c:\clones\clone.2024.09.03.imlst
```
<h2>Show an Open File Dialog</h2>

From IsoBuster 5.5 onwards it is possible to show a dialog that allows you to select a file.<br>
In other words, the filename can be changed while the \*.imlst is being loaded<br>
The syntax is: `\\#\ofd(Title, Filter, Options)c:\path\filename.ext`<br>
Currently only option `m` is supported, which allows you to select more than 1 files *(use with caution as the order of files cannot be guaranteed)*

<h4>Example</h4>

*Triggers a dialog to select a file.  Only \*.img files will be shown, the path will be used*
```
\\#\ofd(Select an IMG file, *.img)c:\path\
```
<h4>Example</h4>

*Triggers a dialog to select a file.  Only \v1\*.im? files in location 'path' will be shown*
```
\\#\ofd(Select a version 1 IMG file, *.img;*.imc)c:\path\v1*.im?
```
<h4>Example</h4>

*Triggers a dialog to select (a) file(s).  Same as previous example, but several files can be selected<br>
Notice the ')' as part of the title inside the brackets is escaped*
```
\\#\ofd(Select one or more version 1 IMG file(s\), *.img, m)c:\path\v1*.img
```
<h4>Example</h4>

*Triggers a dialog to select a file in its simplest form (no title, filter defaults to \*.\*)*
```
\\#\ofd()
```
<h2>Set a path</h2>

From IsoBuster 5.5 onwards.  IsoBuster builds a path based on the location of the \*.imlst file **if** not a full path is provided.
Instead of providing a full path for every individual file you can also set the \\\\path= variable to that path

<h4>Example</h4>

```
\\path=c:\clones\

clone.2024.09.03.imlst
```

![Example of IsoBuster having loaded a *.imlst file that references several other files](Screenshots/test_6.png)
