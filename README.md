[![Build Status](https://travis-ci.org/o2r-project/o2r-meta.svg?branch=master)](https://travis-ci.org/o2r-project/o2r-meta)

# o2r meta

This is a collection of metadata related micro services for the o2r-platform:

0. schema & documentation of the o2r metadata
1. extract - collect meta information from files or session
2. broker - translate metadata from o2r to third party schemas
3. validate - check if metadata set is valid to the schema
4. harvest - collect metadata from external sources via OAI-PMH
 

For their role within o2r, please refer to [o2r-architecture](https://github.com/o2r-project/architecture).

## License

o2r-meta is licensed under Apache License, Version 2.0, see file LICENSE.Copyright (C) 2016, 2017 - o2r project.


## Installation
o2r meta is designed for python 3.6 and supports python 3.4+.

**Installation steps**

(1) Acquire python version 3.4+.

(2) Parts of o2r meta require the _gdal_ module that is known for causing trouble when installed via _PIP_. Therefore it is recommended to prepare the installation like this:

    sudo add-apt-repository ppa:ubuntugis/ppa -y
    sudo add-apt-repository ppa:ubuntugis/ubuntugis-unstable -y
    sudo apt-get -qq update
    sudo apt-get install -y python3-dev
    sudo apt-get install -y libgdal-dev
    sudo apt-get build-dep -y python-gdal
    sudo apt-get install -y python-gdal
    export CPLUS_INCLUDE_PATH=/usr/include/gdal
    export C_INCLUDE_PATH=/usr/include/gdal

and afterwards install _gdal_ this way:

    pip install GDAL==$(gdal-config --version | awk -F'[.]' '{print $1"."$2}')

Alternatively you can use a precompiled python wheel of the gdal module that fits your desired platform.

(3) Install the required modules:

    pip install -r requirements.txt


### Using Docker

Another way of installation is provided by the Dockerfile. Build it like this:

    docker build -t meta .

And start the extractor (e.g.) like this:

	docker run meta o2rmeta.py -debug extract -i extract/tests -o extract/tests -xo

	

## Documentation

+ Current documentation as part of the [ERC-SPEC](http://o2r.info/erc-spec/spec/schema/) ([GitHub](https://github.com/o2r-project/erc-spec/blob/master/docs/spec/schema.md))
+ Current [structure dummy](https://github.com/o2r-project/o2r-meta/blob/master/schema/json/dummy.json)
+ [MD of the erc configuration file](http://o2r.info/erc-spec/spec/#erc-configuration-file)
+ ~~[schema draft](https://raw.githubusercontent.com/o2r-project/o2r-meta/master/schema/json/o2r-meta-schema.json)~~ 

## Usage
When calling o2r meta, you can chose from the following commands, each representing one tool of the o2r meta suite: _extract_, _validate_, _broker_ and _harvest_.

    python o2rmeta [-debug] extract|validate|broker|harvest <ARGS>

Options:

+ `debug` : option to enable verbose debug info where applicable	

Each tool then has a number of required arguments:

#(1) Extractor tool:

	python o2rmeta.py extract -i <INPUT_DIR> -s|-o <OUTPUT_DIR> [-xo] [-m] [-xml] [-ercid <ERC_ID>]
	
Example call:
	
	python o2rmeta.py -debug extract -i extract/tests -o extract/tests -xo
	
Explanation of the switches:

+ `-i` <INPUT_DIR> : required starting path for recursive search for parsable files
+ `-s`: option to print out results to console. This switch is mutually exclusive with `-o`. At least one of them must be given
+ `-o` <OUTPUT_DIR> : required output path, where data should be saved. If the directory does not exist, it will be created on runtime. This switch is mutually exclusive with `-s`. At least one of them must be given.
+ `-xo` : option to disable orcid public API calls
+ `-m` : option to additionally enable individual output of all processed files.
+ `-xml` : option to change output format from json (default) to xml.
+ `-ercid` <ERC_ID>: option to provide an ERC identifier.

### Supported files and formats for the metadata extraction process:

_Feel free to open an issue for suggestions!_

Current version:

**file type** | **description** | **extracted part** | **status**
------ | ------ | ------ | ------ |
.r | R Script | all | implemented
.rmd | R-Markdown | all | implemented
(r session) | live extraction | memory objects | planned
bagit.txt | BagIt | metadata | implemented
.tex | LaTeX | header | planned
.yml | YAML | metadata | planned
.py | python script | all | planned
.shp | Esri shapefile | geometry | implemented
.geojson | GeoJSON | geometry | implemented
.json | GeoJSON | geometry | planned
.tif(f) | geo TIFF | geometry | planned
... | ... | ... | ...


#(2) Brokering/Mapping tool

The broker has two modes: In _mapping mode_, it creates fitting metadata for a given map by following a translation scheme included in that mapping file.
In _checking mode_ it returns missing metadata information for a target service or plattform, e.g. zenodo publication metadata, for a given checklist and input data.

Within o2r, the broker is used to translates between different standards for metadata sets. For example from extracted raw metadata to the o2r schema-compliant metadata. Other target outputs might DataCite XML or Zenodo JSON.
Translation instructions as well as checklists are stored in json formatted files.

    python o2rmeta.py broker -i <INPUT_DIR/FILE> -c <CHECKLIST_FILE>|-m <MAPPING_FILE> -s|-o <OUTPUT_DIR>
	
Example calls:
	
    python o2rmeta.py -debug broker -c broker/checks/zenodo-check.json -i schema/json/example_zenodo.json -o broker/tests/all

    python o2rmeta.py -debug broker -m broker/mappings/o2r-map.json -i broker/tests -o broker/tests/all

Explanation of the switches:


+ `-c` <CHECKLIST_FILE> : required path to a json checklist file that holds checking instructions for the metadata. This switch is mutually exclusive with `-m`. At least one of them must be given.
+ `-m` <MAPPING_FILE> : required path to a json mapping file that holds translation instructions for the metadata mappings. This switch is mutually exclusive with `-c`. At least one of them must be given.
+ `-i` <INPUT_DIR> : path to input json when using `-c`-mode _or_ required starting path for recursive search for parsable files when using `-m`-mode. 
+ `-s`: option to print out results to console. This switch is mutually exclusive with `-o`. At least one of them must be given.
+ `-o` <OUTPUT_DIR> : required output path, where data should be saved. If the directory does not exist, it will be created on runtime. This switch is mutually exclusive with `-s`. At least one of them must be given.

Supported checks/maps


**service** | **checklist file** | **mapping file** | **status**
------ | ------ | ------ | ------ |
zenodo| zenodo-check.json | zenodo-map.json | WIP
datacite| datacite-check.json | datacite-map.json | WIP
... | ... | ... | ...



Governing JSON-Schema for the mapping files: _TBD_


#(3) Validator tool:

	python o2rmeta.py validate -s <SCHEMA> -c <CANDIDATE>
	
Example call:
	
	python o2rmeta.py -debug validate -s schema/json/o2r-meta-schema.json -c schema/json/example1-valid.json

Explanation of the switches:

+ `-s` <SCHEMA> : required path or URL to the schema file, can be json or xml.
+ `-c` <CANDIDATE> : required path to candidate that shall be validated.

#(4) Harvester tool:

Collects OAI-PMH metadata from catalogues, data registries and repositories and parses them to assist the completion of a metadata set such as the one in o2r.

_TBD_
