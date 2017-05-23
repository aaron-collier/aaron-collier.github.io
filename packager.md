# hydra-packager

This chronicles the evolution of a rake task that I wrote for ingesting DSpace data into a hyrax repository.

The main project repo is available in the [hydra-packager repo](https://github.com/aaron-collier/hydra-packager).

## Background

After deciding that Hydra (and specifically Sufia, then Hyrax) would be the platform for the future of the
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

[The code - in all it's glory](https://github.com/aaron-collier/csusm-hyrax/blob/946b9c4497eb88dde5dfa5ac2946024b30dad4db/lib/tasks/packager.rake)

- It works!
- An excellent proof of concept that made possible the move to Hydra
- But, as one of my first ruby projects, this looks like PHP
- Lots of bits commented out, a lot of very local like things, etc...
- Quite a bit that just isn't even used anymore

### Pros

- [The hierarchy that allowed for crawling through the zipfiles](https://github.com/aaron-collier/csusm-hyrax/blob/946b9c4497eb88dde5dfa5ac2946024b30dad4db/lib/tasks/packager.rake#L177)
- [This messy block helped to grok the relationsiop of the data in the mets.xml file](https://github.com/aaron-collier/csusm-hyrax/blob/946b9c4497eb88dde5dfa5ac2946024b30dad4db/lib/tasks/packager.rake#L185)

### Cons

- [No longer calling this method](https://github.com/aaron-collier/csusm-hyrax/blob/946b9c4497eb88dde5dfa5ac2946024b30dad4db/lib/tasks/packager.rake#L260)
- [Hard code work type](https://github.com/aaron-collier/csusm-hyrax/blob/946b9c4497eb88dde5dfa5ac2946024b30dad4db/lib/tasks/packager.rake#L276)
- [Have to manually create this folder](https://github.com/aaron-collier/csusm-hyrax/blob/946b9c4497eb88dde5dfa5ac2946024b30dad4db/lib/tasks/packager.rake#L98)
- Overall, it's just too long and messy.

## Making it usable for anyone else

[I came across this article while investigating rake best practices](https://edelpero.svbtle.com/everything-you-always-wanted-to-know-about-writing-good-rake-tasks-but-were-afraid-to-ask)

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

### Keeping things from hard crashing

- Fixed the bug to create the complete directory
- Added a simple [Error catch](https://github.com/aaron-collier/hydra-packager/blob/master/lib/tasks/packager.rake#L123)

### Minimizing command line options & in code configuration

- Added a [configuration file](https://github.com/aaron-collier/hydra-packager/blob/configuration/config/packager.yml)
- Moved the XML xpath query format into the configuration to clean up...

### Quieting down the output if desired

- Allowed for the existing output amount with at "verbose" option
- Added the "..." style progress bar output as a "minimal" option
- Included a "quite" option for no output.
- All of the above options don't affect the added log output (which matches verbose)

## What's Next?

Having trouble with Pages? Check out our [documentation](https://help.github.com/categories/github-pages-basics/) or [contact support](https://github.com/contact) and we’ll help you sort it out.
