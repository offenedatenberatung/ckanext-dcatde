# ckanext-dcatde

DCAT-AP.de specific CKAN extension for providing and importing DCAT-AP.de-Profile data.

## Dependencies

The CKAN-Plugin ckanext-dcatde is based on the CKAN extension [ckanext-dcat](https://github.com/ckan/ckanext-dcat).

For the RDF harvester, [ckanext-harvest](https://github.com/ckan/ckanext-harvest) is used (optional, see below).

## Getting Started

1. If you are using Python virtual environment (virtualenv), activate it.

2. Install a specific version of the CKAN extension ckanext-dcat. The ckanext-dcatde requires a release greater than `v1.0.0` of ckanext-dcat.

3. Install the extension on your virtualenv:

       (pyenv) $ pip install -e git+git://github.com/GovDataOfficial/ckanext-dcatde.git#egg=ckanext-dcatde
       (pyenv) $ cd src/ckanext-dcatde
       (pyenv) $ pip install -r base-requirements.txt -f requirements
       (pyenv) $ python setup.py develop

4. Install a specific version of the CKAN extension ckanext-harvest. (Only if you want to use the RDF harvester)<br>
It is tested that ckanext-dcatde is working well with the release `v1.1.4` of ckanext-harvest.<br>
   1. Init the harvest tables in the database:

          (pyenv) $ paster --plugin=ckanext-harvest harvester initdb --config=mysite.ini

   2. Create the harvest user

      - create ckan harvest user

            (pyenv) $ paster --plugin=ckan user add harvest password=harvest email=harvest@example.com --config=/etc/ckan/default/production.ini

      - give sysadmin privileges to ckan harvest user

            (pyenv) $ paster --plugin=ckan sysadmin add harvest --config=/etc/ckan/default/production.ini


5. Enable the required plugins by adding to your CKAN configuration file:

       ckan.plugins = dcat dcatde harvest dcatde_rdf_harvester
       
   The plugins `harvest` and `dcatde_rdf_harvester` are only needed if you want to use the RDF harvester.

   In case you like the datasets to be indexed via [Google Dataset Search](https://toolbox.google.com/datasetsearch), activate the `structured_data` plugin in addition. See [ckanext-dcat README](https://github.com/ckan/ckanext-dcat/blob/master/README.md#structured-data-and-google-dataset-search-indexing) for details.


6. Add the following parameter to your CKAN configuration file to activate the additional profile for DCAT-AP.de:

       ckanext.dcat.rdf.profiles = euro_dcat_ap dcatap_de

## RDF DCAT-AP.de Harvester
If the plugin `dcatde_rdf_harvester` is activated an additional source type `DCAT-AP.de RDF Harvester` is selectable.
The harvester supports the additional fields specified in DCAT-AP.de.

### Default license
By default the harvester will set a default license in the resource if in the resource of a dataset is no license
provided. In this case additional information about the harvest source, dataset and resource will be written
as log entry in the info level.

The value which will be used as default license can be defined by the
configuration parameter `ckanext.dcatde.harvest.default_license`. Add the following parameter to your CKAN configuration file, e.g.:

    ckanext.dcatde.harvest.default_license = http://dcat-ap.de/def/licenses/other-closed

### Skipping datasets which does not contain any resources
Skipping datasets which does not contain any resources can be activated by setting the optional
configuration parameter `resources_required` in the harvest source configuration.
Already existent datasets will not be skipped. Add the following parameter into the harvest source
configuration:

    {"resources_required": true}

### Cleaning Tags/Keywords
The DCAT-AP.de profile implements a different logic for cleaning tags/keywords as implemented in ckanext-dcat,
e.g. not replacing/removing German umlauts and 'ß'.


## Creating dcat-ap categories as groups
You need to add the following parameter to your CKAN configuration file:

    ckanext.dcatde.urls.themes = file:///path/to/file/dcat_theme.json

You will find an example file here: [dcat_theme.json](./examples/dcat_theme.json)
If you want to create the standard dcat-ap categories as groups you can use the ckan command "dcatde_themeadder" by following the instructions:

    (pyenv) $ paster --plugin=ckanext-dcatde dcatde_themeadder --config=/etc/ckan/default/production.ini

## Migrating ogd conform datasets to dcat-ap.de
You need to add the following parameter to your CKAN configuration file:

    ckanext.dcatde.urls.license_mapping = file:///path/to/file/dcat_license_mapping.json
    ckanext.dcatde.urls.category_mapping = file:///path/to/file/category_mapping.json

You will find the example files here: [dcat_license_mapping.json](./examples/dcat_license_mapping.json) and [category_mapping.json](./examples/category_mapping.json)
The migration requires that the dcat-ap categories exists as groups in CKAN, see [Creating dcat-ap categories as groups](#creating-dcat-ap-categories-as-groups).
If you want to migrate the datasets from ogd to dcat-ap.de you can use the ckan command "dcatde_migrate" by following the instructions:

    (pyenv) $ paster --plugin=ckanext-dcatde dcatde_migrate --config=/etc/ckan/default/production.ini

With the version 3.1.1 an additional option to the migrate command was added to fix the migration of the OGD field `metadata_original_id`. Instead of mapping this field to `adms:identifier` it will be mapped to the field `dct:identifier` now.
The command can be executed as follows:

    (pyenv) $ paster --plugin=ckanext-dcatde dcatde_migrate adms-id-migrate --config=/etc/ckan/default/production.ini

## Testing

Unit tests are placed in the `ckanext/dcatde/tests` directory and can be run with the nose unit testing framework:

    $ cd /path/to/virtualenv/src/ckanext-dcatde
    $ nosetests
