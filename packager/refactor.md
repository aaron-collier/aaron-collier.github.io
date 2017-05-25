# Refactoring the packager

I may be strange in that I really enjoy refactoring.

The current state of the refactoring is available on the [hydra-packager/refactor branch](https://github.com/aaron-collier/hydra-packager/tree/refactor).

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

## Destroy all the things!

### Goals


### Steps

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
