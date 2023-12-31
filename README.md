# imlst

*.imlst is a file extension, first developed for IsoBuster 5.2 and up.

It is a simple text file that contains a file or physical or logical drive per line.  Paths can be absolute or relative to the position of the *.imlst file iself.
It's an alternative for providing all files/drives on the command line, like so: https://www.isobuster.com/help/open_spanned_files_and_drives and it follows the same logic, so best check out the link.

This text and link serves as specification of the format. Other applications can use it as well, in the exact same way.  To improve sharing *.imlst files between applications, please stick to the same implementation or use a different file extension if you deviate from it.

The extension of the first file in the list can be used by the application that loads it to determine what type file it is.

Example 1 (concatenate two drives as one) :

\\\\.\\PhysicalDrive2<br>
\\\\.\\PhysicalDrive3

Example 2 (treat both files as one file and treat it as a *.dsk file, since that is the extension of the first file):

c:\image file 1.dsk<br>
c:\image file 2.any // comment

An exception to this rule (as implemented by IsoBuster) is when the first file is an *.ibp file.  See above link for more information.
<br>A double forward slash starts a comment, either on a new line, or behind a file/drive

From IsoBuster 5.3 onwards there is also support for a virtual file.  A way to insert virtual data between, after or before file(s).
<br>The syntax is simply: \\\\*\\FileName:(-)Size:Pattern
1. FileName can be any name and must contain at least one character.  It can have a file extension, for instance *.dsk (which would help with proper detection of the image file in case it's the first file)
2. Size must be present and is the number of bytes.  It can be decimal or hexadecimal (in which case it needs to start with 0x or end with h).  If Size is negative, for instance -512, reads in that part of the file will fail with an error.
3. Pattern is a BYTE value and is optional.  Default 0x00 is used but you can specify any value, for instance 0xFF etc.

Example 3 (an iso file is missing 100 blocks and hence all offsets are wrong, fix this by preceding it with a virtual file containing all zeroes):

\\\\*\\.iso:2048000:0x00<br>
c:\file.iso


