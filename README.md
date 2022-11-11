# MunchXMLMuncher 2
## Introduction
Developed by research assistant Loke Sjølie for the University of Oslo

This script takes 1-2 files in eMunch's TEI/XML format and converts them to a complete CMIF/TEIXML file. The script is heavily customized to the project's specifications, and is incredibly unlikely to produce anything worthwhile with files that do not match their exact XML specifications.

The script targets documents that have been tagged with **"brev"** or **"letter"** (excluding, as a rule, drafts), and extracts from these:
1. Document ID, which is extrapolated to form an eMunch URL
2. Document Author, with VIAF ID where applicable
3. Document Authored Date, which is converted to YYYY-MM-DD (or YYYY-MM, or YYYY) or a range that can be from or from-to or to.
4. Document Recipient(s), names and IDs

... and then places these in a hierarchy: <CorrespDesc(DocumentID)><*Author*><*Date*/><*/Author*><*Recipient*(s)/>.

Optional files enable the script to get updated dates from a chronology file and/or placename augmentation. See section *Optional files* below for in-depth explanation.

MXMLM has three parts: the Preprocessor, Core and CMIF Production scripts all work together to create a tailored CMIF file. ~~Alternatively, you can use MXMLM Complete as an all-in-one.~~

### MXMLM Preprocessor
The Preprocessor script scrapes, cleans and transforms data from the optional files to prepare them for use in the core script. The preprocessor script **must** be run before MXML Core if you include any of the optional files. Once run, you do **not** need to run it again until you add, remove or update any of the optional files. See section *Optional files* below.

### MXMLM Core
The Core script scrapes data from the register_tei.xml and/or correspondence.xml files. Please note that these files MUST be named properly for the script to function. The script produces an updated JSON data file that is ready to be converted into CMIF. If the script finds the Preprocessor's output (in the same folder), it'll create an additional data set with any modifications from preprocessing integrated.

### MXMLM Production
The Production script takes the output of Preprocessor and Core and creates a CMIF-compliant XML file.

### MXMLM Complete
~~The Complete script includes all three of the above.~~

## Required variables and metadata
There is a text file by the name config.ini alongside the script. Before running the script, you **must** open this file **in NOTEPAD** (or an appropriate IDE) and do the following:
1. Change the cmifUID to represent this run if applicable (see CMIF documentation for explanation)
2. Change the editorName and editorMail variables - this is you and your eMail
3. Change the publicationStatementFull variable. This variable should contain a "Full bibliographical statement of the scholarly edition or repository where this file points to"
4. Change publisherURL, publisherName, cmifURL, cmifTitle, typeOfBibl as required

The script currently supports a **single** publisher and a **single** editor.

## Required files
To run this script, you will **require** the following in a folder:

A version of register_tei.xml and/or a version of correspondence.xml

The MXMLM script files

(You'll also need Python.)

I **recommend** using Windows. The script has been tested on Windows.

### Optional files (used with MXMLM Preprocessor)
#### Chronology
MXMLM Preprocessor searches for and will use a file named Kronologi_Munchs_brev with the .xlsx filetype if it exists in the same folder as the script (or the source subfolder it creates). As long as the file contains the exact phrase Kronologi_Munchs_brev and is .xlsx, it'll be found (example: Kronologi_Munchs_brev_20220831.xlsx WILL be found, while Kronologi_brev.xlsx will NOT be found). If there are multiple files matching the criteria, the last modified file will be used (possibly Windows-dependent).

If a chronology file is found, the script will index all letters included and replace the dates found in the register/correspondence file with the dates found in the chronology file if such a replacement is possible.

##### Info: Naming conventions
The chronology file's document IDs are consistently inconsistent with the IDs found elsewhere. Thus, the script **expects** to see names in the format "MM N 723", "PN 272" as seen in the provided chronology file. I cannot guarantee that the script will work properly without modification if these IDs are made whole.

##### Info: Date conventions
The script uses a nested decision tree to figure out what sort of date it's looking at in this specific order: datetime, DD.MM.YYYY, MM.YYYY, MM.YYYY/MM.YYYY (range) or MM.YYYY-MM.YYYY (range), YYYY, YYYY-YYYY (range). Dates that do not follow any of these formats are **dropped**. This means that dates that cannot possibly be guessed by a machine without considerable extra work, like "23.07.1895/1896" or "2[6/7].12.1912", are expunged. If the script locates "uncertain" dates expressed by a ? character, the date is trimmed incrementally (D?.MM.YYYY becomes MM.YYYY, ??.YYYY becomes YYYY). If a complete 4-digit long year is not found, the entire date is expunged. Be advised that the script does **not** validate whether the date is between 1860-1950.

Any acceptable date (has at least YYYY, matches a date format that is accounted for) is transformed to match the format YYYY-MM-DD and set as exact date or date range.

#### Placenames
THe Preprocessor searches for a folder named "xml-filer" located in the same folder as itself AND a spreadsheet named "ID_sted-verdier.xlsx". You **must** provide the spreadsheet if you wish to use this functionality, as well as *n* XML files in xml-filer. If found, the script will scan every XML file in that folder (recursive; subfolders are allowed, as long as the top-level folder is named xml-filer) expecting to see a series of letter files. 

The script will search for a dateline element, which may or may not have a placename. The script only looks for a sender because the dateline element is intended to only ever represent the sender's address and date. If the dateline element has a placename element (recursively, again), the script will **assume** that the first @key attribute of this placename element is the address ID of the sender.

If the script is able to locate a placename in the expected format (address, dateline), the value of the @key attribute will be used to compare with ID_sted-verdier.xlsx. The address/place string associated with the @key value is harvested and used as an address element. Otherwise, no place element is appended.

## Simple use case instructions (modular)

1. Place the MXMLM scripts in a folder that contains **at least one** required file (see header REQUIRED FILES).
2. If desired, place optional files and/or folders beside the script (see subheader OPTIONAL FILES). Remember that some options have dependencies.
3. **IMPORTANT!** For placename and date augmentation via XML files and/or chronology file, you MUST run the MXML Preprocessor script **if it has not already been run**. If MXML Preprocessor is not run, MXML Core will *not* create placenames or update dates from a chronology file. It must be run *any time you add, change or remove data from the source XML files, the placename files, or the chronology file(s)*. If you've already run the preprocessor with the current data set, go to next step.
4. Run MXML Core. This is *expected* to take a while - working with XML is extremely slow compared to modern formats.
5. After the script has finished, run MXML Production.
5. The resulting CMIF file is placed in the output subfolder upon script completion.

The MXMLM script will safely copy the register_tei and correspondence XML files used to a separate directory named MXML (version) (date yyyy-mm-dd). If the files are older (by last modified date) than the files it already has in /sourcefiles (created by the script), it will use the NEWER files. To bypass this behaviour, simply delete the sourcefiles folder or place a copy of the script in a flat directory with only the register_tei.xml and correspondence.xml files you wish to use.

## Known bugs and issues
The script does not fetch external UIDs for persons other than Edvard Munch. This would entail using the VIAF API to get IDs on everyone.

The script does not fetch a UID for placenames at time of writing. The CMIF documentation suggests Geonames as an acceptable source of placename UIDs.

The script does not evaluate whether a given date is "certain" or not. It assumes that the dates provided are certain enough (when they form valid dates).

## Known data quality issues
The date formatting in the project's files is not excellent. MM_K4110 is an example of a document that's not registered correctly.

There are many target refs that exist but do not have a value.