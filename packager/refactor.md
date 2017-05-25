# Refactoring the packager

I may be strange in that I really enjoy refactoring.

The current state of the refactoring is available on the [hydra-packager/refactor branch](https://github.com/aaron-collier/hydra-packager/tree/refactor).

## First Steps

### Making it usable for anyone else

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

### Keeping things from hard crashing

- Fixed the bug to create the complete directory
- Added an error directory
- Added a simple [Error catch](https://github.com/aaron-collier/hydra-packager/blob/master/lib/tasks/packager.rake#L123)
- Configuration option to exit on error

### Minimizing command line options & in code configuration

- Added a [configuration file](https://github.com/aaron-collier/hydra-packager/blob/configuration/config/packager.yml)
- Moved the XML xpath query format into the configuration to clean up...
- Allows for addition of metadata properties and work types without touching the rake job

### Quieting down the output if desired

- Allowed for the existing output amount with at "verbose" option
- Added the "..." style progress bar output as a "minimal" option
- Included a "quite" option for no output.
- All of the above options don't affect the added log output (which matches verbose)

## Destroy all the things!

### Goals

- Methods are doing too much, they should do **one** thing
- Stick to DRY principles

### Steps

- Add methods for any current extra method action
- DRY -
    - Remove (when possible) named parameters and pass a params hash

### Example Method

- This method does more than one thing.
    - Creates a directory
    - Extracts files into that directory

```
def unzip_package(zip_file,parentColl = nil)

  zip_file_path = File.join(input_path, zip_file)

  if File.exist?(zip_file_path)
    file_path = File.join(@output_dir, File.basename(zip_file_path, ".zip"))
    @bitstream_dir = file_path
    Dir.mkdir file_path unless Dir.exist?(file_path)
    Zip::File.open(zip_file_path) do |file_to_extract|
      file_to_extract.each do |compressed_file|
        extract_path = File.join(file_path, compressed_file.name)
        zipfile.extract(compressed_file,extract_path) unless File.exist?(extract_path)
      end
    end
  end
end
```

#### First If

```
unless File.exist?(File.join(input_path, params[:source_file])) raise Errno::ENOENT
```

**BUT**

This isn't even necessary, as we _JUST_ checked the existence of this file.

#### Excessive variables

```
params[:unpacked_path] => initialize_directory(File.join(@output_dir, File.basename(params[:source_file], ".zip")))
```
