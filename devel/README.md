# devel files from pre-assembly

## Project Differences

Projects to use as most pressing examples:
- Revs
- SOHP
- Gould
- ReidDennis
- Rumsey [but waiting on specs from Lynn?]

Project structure and differences

- revs
  - one image per digital object
  - flat structure in bundle dir
  - checksums file
  - manifest file with metadata
- rumsey
  - subdir for each digital object
  - one image per object
  - object also includes a descMetadata file
  - no checksums or manifest
  - objects are already registered
  - subdir is the druid
- gould
  - subdir/00 for each digital object
  - multiple images per object
  - images are jpg, not tif
  - objects are already registered
  - subdir is a barcode, which can be used to obtain druid
  - other details:
    - See /thumpers/dpgthumper2-se1/Gould
    - The desc metadata is in symphony.
    - Can get druid using query_by_id(BARCODE) [see dor-services-gem].
    - Content metadata: ask Gary about AP.
    - Misc files at root: probably ignore.
    - Some jpg are in 00 subdir, some are not (36105115575867).
    - Some subdirs have misc files (.zip or .md5). Ignore?
- AP
  - subdir for each digital object
  - within that subdir, several number subdirs: 00 01 02 03 05.
  - different image types/purposes the numbered subdirs
  - multiple images per object
  - multiple types: tif, jp2, and 3 types of jpg
  - objects are already registered
  - main subdir is the druid
- Hummel
  - Same as AP, but fewer number subdirs: 00 05.
- SOHP
  - initially, lab will provide csv manifest
  - later, lab will provide contentMetadata, which might need some transformation
  - subdir for each digital object
  - objects are already registered
  - subdir is the druid
  - descMeta is probably in mdtoolkit
  - contentMetadata contains multiple resource nodes of varying types
    - audio resources
      - there are three types of audio resource nodes: pm, sh, sl
      - there can be multiple resource nodes of each type
      - 3 file nodes per audio resource: audio file; techMeta xml file; md5 text file
    - image resources
      - multiple image resource nodes
      - 2 file nodes per image resource: image file; md5 text file.
    - text resources
      - only 1 resource per object (at least in the example provided)
      - 2 file nodes per image resource: pdf transcript file; md5 text file.
- ReidDennis
  - three parent dirs: tif, jpeg, jp2
  - each parent dir contains multiple images, one image per object
  - if we can ignore the jpeg and jp2 dirs, the project is structured like Revs
  - has a manifest
  - we could create a checksums.txt file
  - manifest requires some more complex templating logic


## Pre-assembly: progress logging and resuming

Add attribute to YAML file:
```yaml
data_for_resume: log/revs_resume.yaml
```

### When run_pre_assemble() is called:
```
If running in resume mode:
    load the persisted data from the prior run
Otherwise:
    truncate the data_for_resume file.
```
### Before pre-assembling individual digital objects:

For each completed step:
- persist any accumulated information
- set an attribute for the step: can_skip_on_resume = true.

Some steps are fast; others take some time:
- discover_objects   # Full crawl of bundle directory.
- load_checksums     # If no provider checksums, create them
- process_manifest   # Fast.
- validate_files     # Gets info from Assembly::ObjectFile.exif for every file.

In process_digital_objects()
```ruby
@digital_objects.each do |dobj|
  # Persist info that we started work on dobj.container.
  begin
    dobj.pre_assemble
    # Persist that we finished dobj.pid.
    puts dobj.pid if @show_progress
  rescue PreAssembly::PreAssembleError => err
    # Persist info about failure, but keep working on other objects.
    # In DigitalObject we should catch survivable exceptions
    # and then throw PreAssembleError.
    # Other -- non survivable -- exceptions should cause pre-assemble
    # to blow up.
  end
end
```
