# Refactoring the packager

I may be strange in that I really enjoy refactoring.

The current state of the refactoring is available on the [hydra-packager/refactor branch](https://github.com/aaron-collier/hydra-packager/tree/refactor).

## First Steps

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
    
### Status

- This method does more than one thing.
-- Creates a directory
-- Extracts files into that directory

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
