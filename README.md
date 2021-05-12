# Strapi plugin import-content

Based off the now broken strapi-plugin-import-content, this plugin attempts to restore the original plugin's functionality, whilst adding support for the new Internationalization feature of Strapi.  Also handles relations, allows for repeated imports of the same records and handles media files from urls found in the import file.

Work in progress, currently only tested with uploaded CSV files, and tailored to the specific needs for Kiddicolour.  Currently the media import also requires an override of the built-in strapi upload service, see below for specific instructions on how to extend the service.

The goal is to remove all project specific customizations so this plugin can be used for any project.


# HERE BE DRAGONS
_This repo is far from production ready, but works optimistically_

Import data into Strapi, as seen on the Strapi Community Blog Posts (4 in total)
https://strapi.io/blog/how-to-create-an-import-content-plugin-part-1-4
This project is written based on the Blog Post above, using the current Strapi methods
and converting to functional React components.

This Blog Post itself is based on the work of https://github.com/jbeuckm/strapi-plugin-import-content
Sadly enough this project seems unmaintained and is not working at all with Strapi
as it currently stands.

An earlier attempt to start off a forked version - https://github.com/4levels/strapi-plugin-import-content
resulted in ... not much

The following additions were done:

### Internationalization support
Imported data containing values in multiple languages are correctly inserted and linked,
allowing the Strapi backend to function properly.

### Relation support
Optimistically try to map related id's contained in a single cell to their related Content Types
by a selectable field, splittable by a configurable separator.
Relations to internationalized records are linked to their translated counterparts.
Handles nested data via parent/children as long as the imported data is sorted properly.
Optionally and limited try to create missing related records.
Works best if the related records are created first though, from leaf to branch so to speak.

### Create and Update support
Allow the selection of an imported field value to be used to match an existing record property,
to allow repeated imports of the same record, updating instead of creating.  Optionally ignore 
records not found in the imported data

### Markdown & URL remapping support
Optionally convert to markdown and parse urls.
Rewrite imported content to accomodate for routing changes.  Currently this is 
hardcoded and tailored to the specific needs of Kiddicolor.

### Fieldmapping guessing
Optimistically guess the field name and language from the imported field names
and the selected target Content Type.

### Date & Workflow support
Validate dates and set created_at and updated_at timestamps of imported records.
If the Draft / Publish feature is enabled, adds the updated_at column as well

## Requirements
### Strapi permissions
Allow public access to the following endpoints
- this plugins endpoints - all ot them
- any custom content type's `find` endpoint
- content-type-builder / getcontenttypes
- i18n / listlocales
- upload / upload

### Relation import
For this to work, the Content Type needs to have an extra column to hold any external id,

### Media import
Is currently broken as there's no way to stream a file directly into Strapi.  See https://github.com/4levels/strapi/commit/4ed25fbdf1d1c2e26bc127c02dc95fd87488814c

#### Extend strapi-plugin-upload
To have the media import working seamlessly (without the need to store the files on the server first) we need a small change in the Strapi Upload service as shown in the related PR here: https://github.com/strapi/strapi/pull/10283
In order to already use this, you can extend this service as follows:

1. create the directory if not existing
   `mkdir -p extensions/upload/services`
2. copy the original service file from Strapi
   `cp node_modules/strapi-plugin-upload/services/Upload.js extensions/upload/services/` 
3. adjust the imports where needed, currently only at line 23
   `const { bytesToKbytes } = require('strapi-plugin-upload/utils/file');`
4. apply the fix from the PR - currently on line 93
  `readBuffer = file && file.buffer || await util.promisify(fs.readFile)(file.path);`

## Caveats - dragons
This code is only tested running locally using Strapi and online using Platform.sh in a 
development environment.
There's definitely issues to be found, this is merely an attempt to make a
usable import feature and move on afterwards as this is a one-off operation.
However since this plugin serves a much wanted feature - almost 700 votes on #23  https://portal.productboard.com/strapi/ - I think more people can benefit from this.
