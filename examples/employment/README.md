 # Scottish Government - Employment data example

This example aims to demonstrate how [table2qb](https://github.com/Swirrl/table2qb) can be used to convert a sample of the [Scottish government's employment data](http://statistics.gov.scot/resource?uri=http%3A%2F%2Fstatistics.gov.scot%2Fdata%2Femployment) from CSV to RDF.


## Directory overview

There are 2 folders in this directory: 

1. [csv](./csv) folder contains the inputs:

    1.1 reference data:
      - components: [components.csv](./csv/components.csv)
      - codelists: [gender.csv](./csv/gender.csv) and [units.csv](./csv/units.csv)

    1.2 observation data:
      - [input.csv](./csv/input.csv)

2. [ttl](./ttl) folder contains the outputs:

    2.1 reference data:
      - components: [components.ttl](./ttl/components.ttl)
      - codelists: [gender.ttl](./ttl/gender.ttl) and [measurement-units.ttl](./ttl/measurement-units.ttl)
     
     2.2 observation data:
      - [cube.ttl](./ttl/cube.ttl)

## Cube structure

The [observations data](csv/input.csv) indicates the required structure of the cube. This cube is a 'measure type' cube with an explicit `qb:measureType` dimension indicating
the measure corresponding to each row. There is a single measure `Count` referenced by the `Measure Type` column. There are four dimension columns (including the `qb:measureType` column):
`Geography`, `Quarter`, `Gender` and `Measure Type`. The `Value` column contains the observed `Count` for each observation, and the `Unit` column is an attribute of the measure which
indicates how to interpret the observed value.
      
## Components

The `Geography` and `Quarter` dimensions use existing vocabularies to define the reference area and period for the observation. The `Measure Type` dimension is also defined by data cube specification.
This leaves only the `Gender` dimension and `Count` measure components which need to be defined. These components are specified within [components.csv](csv/components.csv) and generated by the `components-pipeline`:

    table2qb exec components-pipeline --input-csv examples/employment/csv/components.csv --base-uri http://statistics.gov.scot/ --output-file examples/employment/ttl/components.ttl

## Codelists

The gender dimension defined within [components.csv](csv/components.csv) has an associated codelist. The values of this codelist are defined in [gender.csv](csv/gender.csv) and can be generated using `codelist-pipeline`:

    table2qb exec codelist-pipeline --codelist-csv examples/employment/csv/gender.csv --codelist-name "Gender" --codelist-slug "gender" --base-uri http://statistics.gov.scot/ --output-file examples/employment/ttl/gender.ttl
    
Note the URI of the codelist is generated from the `base-uri` and `codelist-slug` parameters specified above into a URI of the form `{base-uri}/def/concept-scheme/{codelist-slug}`. The resulting URI of `http://statistics.gov.scot/def/concept-scheme/gender`
should match the `Codelist` URI specified for the `Gender` dimension within `components.csv`.

The units of the measure also have an associated codelist defined within [units.csv](csv/units.csv):


    table2qb exec codelist-pipeline --codelist-csv examples/employment/csv/units.csv --codelist-name "Measurement Units" --codelist-slug "measurement-units" --base-uri http://statistics.gov.scot/ --output-file examples/employment/ttl/measurement-units.ttl
    
## Cube

Once the components and codelists have been specified, the cube can be generated from the observation data. The configuration for the input columns is specified in [columns.csv](columns.csv). This defines the
four dimension columns, the `Count` measure and `Unit` attribute as well as the `Value` column. The `Value` column is identified as containing the measure value by its empty `component_attachment`. The
`value_template` of the `Gender` dimension should produce URIs which match the members of the `Gender` concept scheme generated by the `codelist-pipeline` above. Codelist members are identified by URIs of
the form `{base-uri}/def/concept-scheme/{codelist-slug}/{notation}` where `notation` is defined within the codelist CSV file. The `value_template` of `http://statistics.gov.scot/def/concept/gender/{gender}`
for the `Gender` column matches this format. The values of the `Gender` column within the observation data are upper-case i.e. `Female`, `Male`, `All`. These values must be transformed before being substituted
into the value URIs. The `slugize` `value_transformation` specified for the `Gender` column performs this transformation for the values in the input observation data. The cube is then generated with:

    table2qb exec cube-pipeline --input-csv examples/employment/csv/input.csv --dataset-name "Employment" --dataset-slug "employment" --column-config examples/employment/columns.csv --base-uri http://statistics.gov.scot/ --output-file examples/employment/ttl/cube.ttl
    
## Further processing

Given the commands specified above, the resulting data cube will be defined within the [components.ttl](ttl/gender.ttl), [gender.ttl](ttl/gender.ttl), [measurement-units.ttl](ttl/measurement-units.ttl) and
[cube.ttl](ttl/cube.ttl) files.
