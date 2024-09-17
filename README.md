# imlst

*.imlst is a file extension, first developed for IsoBuster 5.2 and up.

<h2>Concatenate files / drives / ..</h2>

Works on real files, virtual files, drives etc.

It is a simple text file that contains a file or physical or logical drive per line.  Paths can be absolute or relative to the position of the *.imlst file iself.
It's an alternative for providing all files/drives on the command line, like so: https://www.isobuster.com/help/open_spanned_files_and_drives and it follows the same logic, so best check out the link.

This text and link serves as specification of the format. Other applications can use it as well, in the exact same way.  To improve sharing *.imlst files between applications, please stick to the same implementation or use a different file extension if you deviate from it.

The extension of the first file in the list can be used by the application that loads it to determine what type file it is.

<h4>Example</h4> 

*concatenate two drives as one*

\\\\.\\PhysicalDrive2<br>
\\\\.\\PhysicalDrive3

<h4>Example</h4> 

*treat both files as one file and treat it as a *.dsk file, since that is the extension of the first file*

c:\image file 1.dsk<br>
c:\image file 2.any // comment

An exception to this rule (as implemented by IsoBuster) is when the first file is an *.ibp file.  See above link for more information.
<br>A double forward slash starts a comment, either on a new line, or behind a file/drive

<h2>Virtual files</h2>

From IsoBuster 5.3 onwards there is also support for a **virtual file**.  A way to insert virtual data between, after or before file(s).
<br>The syntax is: \\\\*\\FileName:(-)Size:Pattern
1. FileName can be any name and must contain at least one character.  It can have a file extension, for instance *.dsk (which would help with proper detection of the image file in case it's the first file)
2. Size must be present and is the number of bytes.  It can be decimal or hexadecimal (in which case it needs to start with 0x or end with h).  If Size is negative, for instance -512, reads in that part of the file will fail with an error.
3. Pattern is a BYTE value and is optional.  Default 0x00 is used but you can specify any value, for instance 0xFF etc.

<h4>Example</h4> 

*an iso file is missing 100 blocks and hence all offsets are wrong, fix this by preceding it with a virtual file containing all zeroes*

\\\\*\\.iso:2048000:0x00<br>
c:\file.iso

<h2>Range in files</h2>

From IsoBuster 5.4 onwards it is also possible to select **a range inside a file, starting from a certain offset**

The syntax is: \\\\#\\(Offset,Range)c:\\path\\filename.ext
1. Offset and Range are in bytes.  This way it is possible to target an embedded file or stitch together a virtual file made up from fragments of one or more files.

<h4>Example</h4> 

*Skip the first 512 bytes, the rest of the file is 1000 512-byte blocks*

\\\\#\\(512,512000)c:\\path\\filename.ext

<h4>Example</h4> 

*Skip the first 512 bytes*

\\\\#\\(512)c:\\path\\filename.ext

<h4>Example</h4> 

*Don't skip anything but there is a range*

\\\\#\\(,512000)c:\\path\\filename.ext

<h4>Example</h4> 

*Stitch 3 files together but skip their 512 byte headers*

\\\\#\\(512)c:\\path\\1.ext<br>
\\\\#\\(512)c:\\path\\2.ext<br>
\\\\#\\(512)c:\\path\\3.ext

To target a zip file that is embedded in a different type file at a certain offset, provide a double range.
The first range works on the unzipped content, the second range tells IsoBuster where the zip file is located in the parent file.

<h4>Example</h4> 

*target a zip file that happens to sit at offset 1000 inside a different file (use virtual file to open the bin as zip)*

\\\\*\\.zip:0|\\\\#\\()\\\\#(1000)c:\\path\\file_with_zip_inside.bin

<h4>Example</h4> 

*target a zip file that happens to sit at offset 1000 inside a different file (use zip prefix to open the bin as zip, see next topic)*

\\\\#\\zip()\\\\#\\()\\\\#(1000)c:\\path\\file_with_zip_inside.bin

<h4>Example</h4> 

*target a zip file that happens to sit at offset 1000 inside a different file but only access 20 bytes at offset 10 inside the decompressed file*

\\\\#\\zip()\\\\#\\(10,20)\\\\#(1000)c:\\path\\file_with_zip_inside.bin

<h2>Target file in zip file</h2>

From IsoBuster 5.5 onwards it is also possible to automatically select **a target** inside a zip file.

Wildcards are supported.  IsoBuster 5.5 onwards supports (most) image file types **including their associated files** inside a zip file.
To explain, in the case of a *.cue and the cue file references one or more *.bin or *.wav or .. files, they can all be located inside the same zip file,
and IsoBuster will find them and open them (in memory, without creating temporary files).

<h4>Example</h4> 

\\\\#\\zip(*.cue)c:\\path\\zipped_cue+iso.zip

Latter example can also exist in this form:

c:\\path\\zipped_cue+iso.(.cue).ibzip

When *.ibzip is associated with IsoBuster you can open it by simply double clicking the file.  If the filename syntax is *.(.ext).ibzip then IsoBuster will look for the first *.ext file inside the zip.
The first dot (\.) inside the brackets is replaced by the wildcard asterix (\*).  If there is no leading dot (\.) the full name must match with a file inside the zip file.
