# MunchXMLMuncher
## Required files
To run this script, you will **require** the following in a folder:

A version of register_tei.xml and/or a version of correspondence.xml

The MXMLM script file

(You'll also need Python)

### Optional files
#### Chronology
The script searches for and will use a file named Kronologi_Munchs_brev with the .xlsx filetype if it exists in the same folder as the script (or the source subfolder it creates). As long as the file contains the exact phrase Kronologi_Munchs_brev and is .xlsx, it'll be found (example: Kronologi_Munchs_brev_20220831.xlsx WILL be found, while Kronologi_brev.xlsx will NOT be found). If there are multiple files matching the criteria, the last modified file will be used (THIS MAYBE/PROBABLY REQUIRES WINDOWS).

If a chronology file is found, the script will index all letters included and replace the dates found in the register/correspondence file with the dates found in the chronology file if such a replacement is possible.

#### Placenames
The script searches for a folder named "xml-filer" located in the same folder as itself AND a spreadsheet named "ID_sted-verdier.xlsx". You **must** provide the spreadsheet if you wish to use this functionality. If found, the script will scan every XML file in that folder (recursive; subfolders are allowed, as long as the top-level folder is named xml-filer) expecting to see a series of letter files. 

Here, the script looks for **address** elements. Acceptable attributes are @recipient and @sender. Instances where the attribute(s) differ from recipient/sender or an attribute is missing are missing are ignored. There can be multiple recipients and/or multiple senders. There can be multiple address @sender elements referring to the same sender. There may or may not be a name/ID for the sender/recipient in the address element(s). There can be several placenames in each address element. There may not be an address in the address element. There may be an organization assigned as an address - these are ignored because their IDs do not correspond to anything in the provided ID>string translation spreadsheet. The script searches recursively (required because there is *n* addressline elements between any given address and its placename(s)) through the child elements of the address element until it locates a placename with an @key attribute to harvest.

The XML files also (sometimes) have a **dateline** element. If the script finds an address for the sender in the address element(s), the dateline element is ignored. If sender address is missing entirely, the script will search for a dateline element, which may or may not have a placename. The script only looks for a sender because the dateline element does not assign a sender or recipient attribute, and I will therefore assume that the dateline refers to the sender. If the dateline element has a placename element (recursively, again), the script will assume that the key attribute of this placename element is the address of the sender.

If the script is able to locate a placename in the expected format (address, dateline), the value of the @key attribute will be used to compare with ID_sted-verdier.xlsx. The address/place string associated with the @key value is harvested and used as an address element. Otherwise, the address field(s) are left empty.

The user is advised that extracting such irregular and inconsistent metadata from 13 000+ files on the fly is complicated. Until the resulting dataset is thoroughly examined, I do not believe nor advise that it should be interpreted as correct. Fact-checkers: I expect that any factual errors will be most obvious in letters with multiple recipients and/or senders, so start there.

## Simple use case instructions
1. Place the MXMLM script in a folder that contains **at least one** required file (see header REQUIRED FILES).
2. If desired, place optional files and/or folders beside the script (see subheader OPTIONAL FILES). Remember that some options have dependencies.
3. Run the script.
4. Wait for 4-10 minutes (hardware dependent).
5. The resulting CMIF file is placed in the output subfolder upon script completion.

The MXMLM script will safely copy the register_tei and correspondence XML files used to a separate directory named MXML (version) (date yyyy-mm-dd). If the files are older (by last modified date) than the files it already has in /sourcefiles (created by the script), it will use the NEWER files. To bypass this behaviour, simply delete the sourcefiles folder or place a copy of the script in a flat directory with only the register_tei.xml and correspondence.xml files you wish to use.

## Known data quality issues
The date formatting in the correspondence file is straight up not good. It is unclear if a date registered as "uncertain" is more or less uncertain than an uncertain date in sharp brackets. MM_K4110 is an example of a document that's just not registered correctly. 

The correspondence file does not use the same coding ("author" vs "sender") as the register_tei file. Applies to dates as well.

Plenty of target refs with no ref value, particularly in correspondence.

### XML files
The coding of addresses and recipients in the source XML files is inconsistent. Addresses *might* be coded as a "dateline" or an "address" element with one or more placenames - or not at all.