# MunchXMLMuncher 1.0 "Pescetarius" Instructions
## Introduction
Developed by research assistant Loke Sjølie for the University of Oslo

This script takes 1-2 files in eMunch's TEI/XML format and converts them to a complete CMIF/TEIXML file. The script is heavily customized to the project's specifications, and is incredibly unlikely to produce anything worthwhile with files that do not match their exact XML specifications. Be aware that CMIF is purely intended to represent correspondance between individuals, and as such there is *significant* (intentional) data loss in converting to the format.

The script targets documents that have been tagged with **"brev"** or **"letter"** (excluding, as a rule, drafts), and extracts from these:
1. Document ID, which is extrapolated to form an eMunch URL
2. Document Author, with VIAF ID where applicable
3. Document Authored Date, which is converted to YYYY-MM-DD (or YYYY-MM, or YYYY) or a range that can be from or from-to or to.
4. Document Recipient(s), names and IDs

... and then places these in a hierarchy: <CorrespDesc(DocumentID)><*Author*><*Date*/><*/Author*><*Recipient*(s)/>.

Optional files enable the script to get updated dates from a chronology file and/or placename augmentation. See header Optional files below for in-depth explanation.

## Required variables and metadata
Before running this script, you **must** do the following:
1. Change the cmifUID to represent this run if applicable
2. Change the editorName and editorMail variables - this is you
3. Change the publicationStatementFull variable. This variable should contain a "Full bibliographical statement of the scholarly edition or repository where this file points to"
4. Change publisherURL, publisherName, cmifURL, cmifTitle, typeOfBibl as required

The script currently supports a **single** publisher and a **single** editor. All options are found at the top of the script.

## Required files
To run this script, you will **require** the following in a folder:

A version of register_tei.xml and/or a version of correspondence.xml

The MXMLM script file

(You'll also need Python.)

I **recommend** using Windows. The script has been tested on Windows.

### Optional files
#### Chronology
The script searches for and will use a file named Kronologi_Munchs_brev with the .xlsx filetype if it exists in the same folder as the script (or the source subfolder it creates). As long as the file contains the exact phrase Kronologi_Munchs_brev and is .xlsx, it'll be found (example: Kronologi_Munchs_brev_20220831.xlsx WILL be found, while Kronologi_brev.xlsx will NOT be found). If there are multiple files matching the criteria, the last modified file will be used (possibly Windows-dependent).

If a chronology file is found, the script will index all letters included and replace the dates found in the register/correspondence file with the dates found in the chronology file if such a replacement is possible.

##### Info: Naming conventions
The chronology file's document IDs are consistently inconsistent with the IDs found elsewhere. Thus, the script **expects** to see names in the format "MM N 723", "PN 272" as seen in the provided chronology file. I cannot guarantee that the script will work properly without modification if these IDs are made whole.

##### Info: Date conventions
At present, the chronology file's dates follow no convention and adhere to neither rules or law. There are more than ten different formats in use, some of which are more sensible than others. The script uses a nested decision tree to figure out what sort of date it's looking at in this specific order: datetime, DD.MM.YYYY, MM.YYYY, MM.YYYY/MM.YYYY (range) or MM.YYYY-MM.YYYY (range), MM.-MM.YYYY (range), YYYY, YYYY-YYYY (range). Dates that do not follow any of these formats are **dropped**. This means that dates that cannot possibly be guessed by a machine without considerable extra work, like "23.07.1895/1896" or "2[6/7].12.1912", are expunged. At time of writing, this excludes 20 dates. If the script locates "uncertain" dates expressed by a ? character, the date is trimmed incrementally (D?.MM.YYYY becomes MM.YYYY, ??.YYYY becomes YYYY). If a complete 4-digit year is not found, the entire date is expunged. Be advised that the script does **not** check if the date is between 1860-1950.

Any acceptable date (has at least YYYY, matches a date format that is accounted for) is transformed to match the format YYYY-MM-DD and set as exact date or a date range.

#### Placenames
The script searches for a folder named "xml-filer" located in the same folder as itself AND a spreadsheet named "ID_sted-verdier.xlsx". You **must** provide the spreadsheet if you wish to use this functionality, as well as *n* XML files in xml-filer. If found, the script will scan every XML file in that folder (recursive; subfolders are allowed, as long as the top-level folder is named xml-filer) expecting to see a series of letter files. 

A (currently disabled) option sets the script to look for **address** elements. Acceptable attributes are @recipient and @sender. Instances where the attribute(s) differ from recipient/sender or an attribute is missing are missing are ignored. There can be multiple recipients and/or multiple senders. There can be multiple address @sender elements referring to the same sender. There may or may not be a name/ID for the sender/recipient in the address element(s). There can be several placenames in each address element. There may not be an address in the address element. There may be an organization assigned as an address - these are ignored because their IDs do not correspond to anything in the provided ID>string translation spreadsheet. The script searches recursively (required because there is *n* addressline elements between any given address and its placename(s)) through the child elements of the address element until it locates a placename with an @key attribute to harvest. **At the moment, this option is disabled by default due to the inability to guarantee that the results are accurate.**

Some XML files have a **dateline** element. ~~If the script finds an address for the sender in the address element(s), the dateline element is ignored. If sender address is missing entirely,~~ the script will search for a dateline element, which may or may not have a placename. The script only looks for a sender because the dateline element is intended to only ever represent the sender's address and date. If the dateline element has a placename element (recursively, again), the script will assume that the first @key attribute of this placename element is the address ID of the sender.

If the script is able to locate a placename in the expected format (address, dateline), the value of the @key attribute will be used to compare with ID_sted-verdier.xlsx. The address/place string associated with the @key value is harvested and used as an address element. Otherwise, no place element is appended.

## Simple use case instructions
1. Place the MXMLM script in a folder that contains **at least one** required file (see header REQUIRED FILES).
2. If desired, place optional files and/or folders beside the script (see subheader OPTIONAL FILES). Remember that some options have dependencies.
3. Run the script.
4. Wait 8~12 minutes.
5. The resulting CMIF file is placed in the output subfolder upon script completion.

The MXMLM script will safely copy the register_tei and correspondence XML files used to a separate directory named MXML (version) (date yyyy-mm-dd). If the files are older (by last modified date) than the files it already has in /sourcefiles (created by the script), it will use the NEWER files. To bypass this behaviour, simply delete the sourcefiles folder or place a copy of the script in a flat directory with only the register_tei.xml and correspondence.xml files you wish to use.

## Known bugs and issues
The script does not fetch external UIDs for persons other than Edvard Munch. This would entail using the VIAF API to get IDs on everyone.

The script does not fetch a UID for placenames at time of writing. The CMIF documentation suggests Geonames as an acceptable source of placename UIDs.

The script does not evaluate whether a given date is "certain" or not. It assumes that the dates provided are certain enough (when they form valid dates).

The script disagrees heavily with the way dates are annotated in the correspondence.xml file and will report over 100 data errors when this file is in use.

### Todo
Further improvement may include placing the harvested placenames, dates and so forth in a file after running the script to facilitate quicker re-runs. However, this may confuse some users, and ease of use generally trumps speed.

## Known data quality issues
The date formatting in the project's files is not excellent. MM_K4110 is an example of a document that's not registered correctly.

There are many target refs that exist but do not have a value.