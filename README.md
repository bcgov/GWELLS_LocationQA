# GWELLS location data QA

Python script that attempts to identify records in GWELLs having coordinates that need attention.

## Installation/requirements

Installation of the script and requirements is easiest via `conda`:

    git clone https://github.com/smnorris/gwells_locationqa
    cd gwells_locationqa
    conda env create -f environment.yml
    conda activate gwells_locationqa

## Download ESA WorldCover 2020

Most required data are downloaded by the script but the ESA WorldCover download is a separate task.
To download tiff tiles for BC, merge, and warp to BC Albers, ensure `awscli` and `gdal` are available at the command line and run the script:

    ./get_esa_worldcover_bc.sh

The provided script is for `bash` or similar but can be modified to run on windows with only a couple of changes.

For more info on the WorldCover dataset:

- [https://esa-worldcover.org/en](https://esa-worldcover.org/en)
- [https://esa-worldcover.s3.amazonaws.com/readme.html](https://esa-worldcover.org/en)


## Usage

Run the job in several steps:

    python gwells_locationqa.py download  # download required data to /data folder (gwells and pmbc parcel fabric)
    python gwells_locationqa.py geocode <GEOCODER_API_KEY>  # reverse-geocode all wells (requires ~6hrs)
    python gwells_locationqa.py qa        # match wells pids to pmbc, match addresses, overlay with agriculture data sources


## Output

Output file is `gwells_locationqa.csv`. All wells with a lat/lon are included.

Output columns are as per the source `well.csv` file, with the following additions:


| Column                       | Description   |
| ----------------             | ------------- |
| nr_district_name             | Natural Resource District
| nr_region_name               | Natural Resource Region
| fullAddress                  | geocoder result
| streetAddress                | geocoder results parsed to match gwells address
| civicNumber                  | geocoder result
| civicNumberSuffix            | geocoder result
| streetName                   | geocoder result
| streetType                   | geocoder result
| isStreetTypePrefix           | geocoder result
| streetDirection              | geocoder result
| isStreetDirectionPrefix      | geocoder result
| streetQualifier              | geocoder result
| localityName                 | geocoder result
| distance_geocode             | distance from well to result of geocode (value 99999 indicates no result)
| distance_to_matching_pid     | distance from well to BC Parcel Fabric polyon with matching PID (0 indicates the well point intersects the matching PID polygon, NULL indicates no matching PID found)
| score_address                | [Token Set Ratio](https://github.com/seatgeek/thefuzz) score for matching wells address to reverse geocoded address (street number/name/direction)
| score_location_description   | [Token Set Ratio](https://github.com/seatgeek/thefuzz) score for matching wells location description to reverse geocoded full address
| score_city                   | [Token Set Ratio](https://github.com/seatgeek/thefuzz) score for matching wells city to reverse geocoded locality
| xref_ind                     | Indicates if one of these strings found in comments column: `CROSS R,CROSS-R,REF'D,REFERENCED,REFD,XREF,X-R,X R`
| alr_ind                      | Indicates if the well is within the ALR as defined by [WHSE_LEGAL_ADMIN_BOUNDARIES.OATS_ALR_POLYS](https://catalogue.data.gov.bc.ca/dataset/alc-alr-polygons)
| btm_label                    | [BTM](https://catalogue.data.gov.bc.ca/dataset/baseline-thematic-mapping-present-land-use-version-1-spatial-layer) `PRESENT_LAND_USE_LABEL` at well location
| esa_landclass                | [ESA WorldCover](https://esa-worldcover.org/en) land class at well location

See `notebooks/locationqa.ipynb` for an example of how to work through the various address matching scores.