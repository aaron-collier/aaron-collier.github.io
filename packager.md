# hydra-packager

This chronicles the evolution of a rake task that I wrote for ingesting DSpace data into a hyrax repository.

The main project repo is available in the [hydra-packager repo](https://github.com/aaron-collier/hydra-packager).

## Background

After deciding that Hydra (and specifically Sufia, and now Hyrax) would be the platform for the future of the
California State University ScholarWorks project, the first order of business was determining what the movement
of data out of DSpace would look like.

As a simple primer, there are several options this:
- METS export
- [AIP export](https://wiki.duraspace.org/display/DSDOC3x/DSpace+AIP+Format)
- Bag plugin
- OAI-PMH
- Raw database/SOLR/bitstreams

In order to avoid installing a plugin (Bag) and wanting to stay as close to native DSpace as possible, I chose to use the results of the AIP packager for Hydra ingestion. This has the benefit of being the prescribed backup/restore method for DSpace.

## Version 1 (or 0.1 if you prefer)

With that decision made, getting data out of DSpace was easy, now I needed to take that data and injest it into hyrax.

### AIP Structure

```
filename.zip
-- mets.xml
COMMUNITY@1234-5678.zip
-- mets.xml
COLLECTION@2345-6789.zip
-- mets.xml
-- bitstream_1234.jpg*
ITEM@0987-1234.zip
-- mets.xml
-- bitstream_1234.txt
-- bitstream_1234.pdf
-- bitstream_1234.jpg
```

Exported file and COMMUNITY zip files generally only contain mets.xml (and possibly a thumbnail). The mets.xml file includes links the the sub community and collection zip files.

Collection zip files contain a mets.xml file, possibly a thumbnail. The mets.xml file includes the structure to the ITEM files.

The ITEM files contain the metadata (in mets.xml) and any associated bitstreams. These are the objects we want imported.

[The code - in all it's glory](https://github.com/aaron-collier/hydra-packager/blob/v1.0/lib/tasks/packager.rake)

- It works!
- An excellent proof of concept that made possible the move to Hydra
- But, as one of my first ruby projects, this looks like PHP
- Lots of bits commented out, a lot of very localized bits, odd names, etc.
- Quite a bit that just isn't even used anymore
- Not at all DRY
- Methods tend to do too much


## Making it usable for anyone else

[I came across this article while investigating rake best practices](https://edelpero.svbtle.com/everything-you-always-wanted-to-know-about-writing-good-rake-tasks-but-were-afraid-to-ask)

- Added a description, this makes it show up in `rake -T`
- Added a method to print a progress bar (but admittedly this compounded the DRY problem)
    - Adding a log service to both minimize output and DRY between log and console output.

### Write the docs!

[Sent this gist to Drew](https://gist.github.com/aaron-collier/de0d5e885813aecc4b755ca634745e09)

With this particular beautifully complete documentation:

```
# Steps to use this:
# 1 - Export DSpace data in AIP format
# ---- [dspace bin]/dspace packager -d -a -e [email address] -i [handle of comm/coll/item] -t AIP [full path to export file name in .zip format]
# ---- this will include all sub items and collections in ITEM-HANDLE.zip format - move all files to server for import
# 2 - in the directory with the above zip files, add a "complete" directory (this should be added to the code, just hasn't been done yet)
# 3 - run the rake from your hydra project root as: rake packager:aip["path/to/top_level_zip","admin@somehere.edu"] (where admin@ is your admin email address)

# a few things to keep in mind, the below "attributes" has is largely dependant on our data mapping, so if those dublin core fieds show up
# you'll need to comment them out to not include them, or add them to your model. the attribute is based on the dc key.

# There's a bit here that I'm not doing anything with anymore or yet, like capturing the community heirarchy to include in metadata. should be easy to reistablish that

# Sometimes a dspace created zip file will cause an error. Remove or move that file then move your NON item zip files back from "complete
 to the root folder and rerun to catch up from where it failed.
 ```

At this point, it was very clear that a [refactor](packager/refactor) was necessary.

## TODO & What's Next?

- Improve the README and any other instructions
- Gem for easy inclusion in a project
- Investigate ability to attach files without running derivatives (for the sake of speed)
- Consider options for automation
- Make the task interactive so they user can select the input file without a command line option
