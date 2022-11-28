# MunchXMLMuncher 2
Developed by research assistant Loke Sjølie for the University of Oslo
## Changelog
### Version 2 pre-release (current)
1. **Extreme** overall performance increase! Running the script went from taking 18 minutes to 30 seconds.
2. New and more accurate method of constructing CMIF file.
3. Fixed the dummy and empty elements bugs.
TODO: clean up and prepare an ordinary .py script that runs without fuss.

### Version 2 alpha
1. Significant performance and completeness increase in fetching placenames
2. Significantly increased number of dates included from the chronology, **with some caveats**: the chronology file now requires mostly-correct document/object ID formatting. See optional files - chronology for details.
3. Combined Preprocessor and Core scripts for ease of use
4. Integrated place and date augmentation into JSON data creation for correspondence and register_tei

## Introduction
This script takes 1-2 files in eMunch's TEI/XML format and converts them to a complete CMIF/TEIXML file. The script is heavily customized to the project's specifications, and is incredibly unlikely to produce anything worthwhile with files that do not match their exact XML specifications.

The script targets documents that have been tagged with **"brev"** or **"letter"** (excluding, as a rule, drafts), and extracts from these:
1. Document ID, which is extrapolated to form an eMunch URL
2. Document Author, with VIAF ID where applicable
3. Document Authored Date, which is converted to YYYY-MM-DD (or YYYY-MM, or YYYY) or a range that can be from or from-to or to.
4. Document Recipient(s), names and IDs

... and then places these in a hierarchy: <CorrespDesc(DocumentID)><*Author*><*Date*/><*/Author*><*Recipient*(s)/>.

Optional files enable the script to get updated dates from a chronology file and/or placename augmentation. See section *Optional files* below for in-depth explanation.

MXMLM has three parts: the Preprocessor, Core and CMIF Production scripts all work together to create a tailored CMIF file.
### MXMLM Complete
From version 2 onwards, MXMLM has reintegrated all three parts described below.

#### MXMLM Preprocessor
The Preprocessor script scrapes, cleans and transforms data from the optional files to prepare them for use in the core script. The preprocessor script **must** be run before MXML Core if you include any of the optional files. Once run, you do **not** need to run it again until you add, remove or update any of the optional files. See section *Optional files* below.

#### MXMLM Core
The Core script scrapes data from the register_tei.xml and/or correspondence.xml files. Please note that these files MUST be named properly for the script to function. The script produces an updated JSON data file that is ready to be converted into CMIF. If the script finds the Preprocessor's output (in the same folder), it'll create an additional data set with any modifications from preprocessing integrated.

New in version 2: these have been effectivised and now run at extremely high superspeeds.

#### MXMLM Production
The Production script takes the output of Preprocessor and Core and creates a CMIF-compliant XML file.

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
**IMPORTANT**: when using the chronology file, ensure that ALL object IDs are correct and compliant. This means that all object IDs MUST follow one of two formats. These are the 6-character PN objects (PN1234), and the 11-character No objects (No-MM_X1234). You are permitted to leave or remove leading zeroes following the full prefix (PN or No-MM_X).

In cases where the object ID is too short (PN50), zeroes (0) are added immediately after the prefix until it complies (PN0050). In cases where the object ID is too long (No-MM_X012345678) and there is a comma, **everything** to the right of the comma is expunged. If the string is too long *and* the string has a No-prefix, characters matching 0 are removed from immediately after the prefix until it complies (this is common). Once the prefix is not followed by 0 or the string is 11 characters long, characters are removed from the end of the string until it complies (resulting in No-MM_X1234). The string is then checked for compliance: it must be 6 or 11 characters long, and the last 4 characters must be numeric.

##### Info: Date conventions
The script instantly accepts datetime-formatted dates. If the script locates "uncertain" dates expressed by a ? character, the date is trimmed incrementally (D?.MM.YYYY becomes MM.YYYY, ??.YYYY becomes YYYY). If a date contains the character -, it is assigned as a range date and is split in from and to dates. The script will correct from-to dates where the to date is more specific than the from date, as in 11-12.2000 -> 11.2000-12.2000. *Note that the script will alert but **not** correct date ranges where the from date is more specific, as in 11.2000-12. This is a feature, not a bug.* Date ranges are stored in the format FROM%TO. If the date does not contain the character -, it is formatted as a single-instance date (YYYY.MM.DD). 

All dates are fetched whole, including the day and/or the month wherever possible: the minimum requirement is a 4-digit year. If a complete 4-digit long year is not found, the entire date is expunged. Be advised that the script does **not** validate whether the date is between 1860-1950.

#### Placenames
The Preprocessor searches for a folder named "xml-filer" located in the same folder as itself AND a spreadsheet named "ID_sted-verdier.xlsx". You **must** provide the spreadsheet if you wish to use this functionality, as well as *n* XML files in xml-filer. If found, the script will scan every XML file in that folder (recursive; subfolders are allowed, as long as the top-level folder is named xml-filer) expecting to see a series of letter files. 

The script will search for a dateline element, which may or may not have a placename. The script only looks for a sender because the dateline element is intended to only ever represent the sender's address and date. If the dateline element has a placename element (recursively, again), the script will assume that the first @key attribute of this placename element is the address ID of the sender.

If the script is able to locate a placename in the expected format (address, dateline), the value of the @key attribute will be used to compare with ID_sted-verdier.xlsx. The address/place string associated with the @key value is harvested and used as an address element. Otherwise, no place element is appended.

## Simple use case instructions (modular)
1. Place the MXMLM scripts in a folder that contains **at least one** required file (see header REQUIRED FILES).
2. If desired, place optional files and/or folders beside the script (see subheader OPTIONAL FILES). Remember that some options have dependencies. Ensure that the chronology file has correctly formatted object/document IDs.
3. Run MXMLM via python.
4. The resulting CMIF file is placed in the output subfolder upon script completion.

## Known bugs and issues
The script does not fetch external UIDs for persons other than Edvard Munch. This would entail using the VIAF API to get IDs on everyone.

The script does not fetch a UID for placenames at time of writing. The CMIF documentation suggests Geonames as an acceptable source of placename UIDs.

The script does not evaluate whether a given date is "certain" or not. It assumes that the dates provided are certain enough (when they form valid dates).

## Known data quality issues
MM_K4110.

There are many target refs that exist but do not have a value.