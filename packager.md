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

#### examples

[Example Collection METS](packager/examples/collection_mets.xml)

[Example Item METS](packager/examples/item_mets.xml)

#### Code

[The code - in all it's glory](https://github.com/aaron-collier/hydra-packager/blob/v1.0/lib/tasks/packager.rake)

- It works!
- An excellent proof of concept that made possible the move to Hydra
- But, as one of my first ruby projects, this looks like PHP
- Lots of bits commented out, a lot of very localized bits, odd names, etc.
- Quite a bit that just isn't even used anymore
- Not at all DRY
- Methods tend to do too much

At this point, it was very clear that a [refactor](packager/refactor) was necessary.

## TODO & What's Next?

- Improve the README and any other instructions
- Gem for easy inclusion in a project
- Investigate ability to attach files without running derivatives (for the sake of speed)
- Possible notifications for long running tasks
- Consider options for automation
- Make the task interactive so they user can select the input file without a command line option
