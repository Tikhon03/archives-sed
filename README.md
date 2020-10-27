# Archives-sed
A script for converting MS word inventories to EAD format

### Do we really need another word to EAD converter?
Yes. The problem is that in many cases the files being converted to EAD,
such as MS word documents, do not have much formatting or
structure. Whatever structure the document does have many be inconsistent,
varying not only from institution to institution,
but also from collection to collection. Harvard has provided an
[excel spreadsheet](https://github.com/harvard-library/aspace-import-excel)
as an aid for constructing EAD files. However, you somehow need to add the
entries to the spreadsheet. Harvard does not seem to provide an automated
way of populating the spreadsheet, and it is not clear that adding entries
to the spreadsheet manually is any faster than manually creating an EAD file
 without using the spreadsheet at all (by copying the 
 inventory into a text file and manually adding the EAD tags 
 where they belong). The methods provided by ArchivesSpace are not much better.

### Will the Archives-sed script work for my word document?
It all depends on how your document is structured. If you have a word document
with tables, then you may want to use
[ArchivesR](https://github.com/ameyer24/ArchivesR) instead.
If your word document does not have tables though, then the R code in ArchivesR 
will find nothing. If your word documents are structured by indenting with tab,
as in the sample [inventory file](GriffithInventory.docx), then the Archives-sed
script will probably work with minor changes.
If your word document is structured using various textual indicators, such as
bold, italics, underlines, centering, or alphanumeric codes, then the script
will probably work with more significant changes.
If you are starting with an html file instead of a word document, then it may
still be possible to adapt the script to work as well. Please see the section
below on customizing the script for more information.

### How do I use the script?
The script is a bash script using [gnu sed](http://www.gnu.org/software/sed/).
It is currently *not* compatible with bsd sed, and therefore would need to be
changed before it would be truly cross platform. The following instructions assume that
you are using some version of Linux.

1. Download the script. This can be done either by git clone, or by downloading
just the [script file](EAD_script).
2. Either move the script file to the folder containing the inventories, or move
the inventories to the folder containing the script file. Copying or creating a symbolic link 
are possible as well.
3. Double check that the script file is executable. If not, then change the permissions.
The permissions can be viewed and changed from a file browser or from the command line. From the command line:  
```
$ ls -l EAD_script
```  
```
$ chmod +x EAD_script
```  
4. Execute the script:  
```
$ ./EAD_script
```
5. Open the output xml files in a text or code editor, and do any necessary cleanup before uploading to ArchivesSpace. Inconsistencies in the formatting of the word documents read by the script can cause various issues to occur including
    - Excessive nesting of <c01-07\> and </c01-07\> tags.
    - Missing title or date in a <did\> entry. ArchivesSpace requires title or date for each entry, for xml import to work.
    - Missing closing tags. Use find (ctrl-F) to count the opening and closing tags of each type. If for some reason the number of opening and closing tags is not the same, then find can be used to help locate the opening tags without a corresponding closing tag. ArchivesSpace will not import any file for which an opening tag is missing its corresponding closing tag.
6. Import the xml file in ArchivesSpace.

If you compare GriffithInventory.xml, which is the raw output of the script when applied to GriffithInventory.docx, with GriffithInventory-final.xml, you will see what the final result should look like after cleanup. The resources folder provides samples from ArchivesSpace and from the EAD website that also serve as useful examples of a correctly formatted EAD file. Note that the semi-colon in the ArchivesSpace sample is not necessary.

### Customizing the script

A docx file contains xml formating already. EAD_script begins by extracting the xml from the docx file (see the for loop at the end of the file). It then finds the relevant tags in the file (<w:tab\> for tab, <w:b...\> for bold, etc.) and creates corresponding flags (\_tab\_, \_b\_, etc.) and then discards the tags. So, if your docx file uses bold, italics, centering, etc., in a different way than the sample, you should begin by changing script1 (at the beginning of the file) to search for the correct tags and convert them to appropriate flags before the tags are discarded. 

Next, script1 uses the flags to identify the the <c\> level by indentation, and the titles for each <did\> entry. In the case of the sample inventory provided, the SC number is even more useful than tab, bold, etc., so if your docx file uses alphanumeric identifiers in a similar way, you can adjust the script to identify those.

After removing blank lines with grep, script2 identifies the date and the physical description for each <did\> entry, and finishes formatting the EAD file. If different date formatting conventions are used, then the regular expression used to identify the date in script2 will need to be revised. You should observe that <c07\> is the maximum <c\> level supported by the code, while EAD specifications allow up to <c12\>. If your file requires anything higher than <c07\> you will need to extend the pattern to the required level.

In the for loop itself, you need to make sure that f ranges over whatever file name pattern you are using, including the file type (docx, html, etc.). If you are trying to convert an html file, then you do not need the extraction step, so that line of the for loop should be commented out. The iconv line is currently commented out. You should uncomment that line if you want foreign characters to be transliterated. Note that ArchivesSpace will accept the foreign characters, but searching is sometimes easier with transliterations. So, I would say it is a matter of taste.
Personally, I think that the output files should be Inventory.ead, but xml is a standard suffix and ead currently is not.

