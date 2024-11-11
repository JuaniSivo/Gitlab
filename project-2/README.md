# GIS Project "ITRF2020"

The International Terrestrial Reference Frame (ITRF) is computed from geodetic data collected at observatories all over the world. The coordinates of the permanent instruments at those sites are quality controlled thanks to the availability of the relative position of these instruments determined with standard surveying techniques. Unfortunately, those observations are expensive and rarely repeated. A working group of the International Association of Geodesy is questioning the use of InSAR technology to monitor the relative displacements of the instruments at those sites.

InSAR analysis is a specific processing that makes use of Synthetic Aperture Radar (SAR) space images of the same area to compute a deformation map with mm/yr accuracy. During the lifetime of a SAR satellite, many SAR images are acquired around the world as a function of the user needs. Thus, we would like to make the inventory of  all SAR images collected from Sentinel 1A and Sentinel 1B satellites that cover each ITRF site in order to investigate if deformation maps could be processed. Indeed, at least 12 images of the same area acquired from the same acquisition geometry (same “relative orbit number”) are required to obtain reliable InSAR results.

We focus on ITRF sites that include at least 2 instruments from 2 different measurement techniques (GNSS, DORIS, SLR or VLBI). Two instruments are located on the same site if their DOMES number (ID number of the instrument) starts with the same 5 numbers.

The objective of this project is to collect and map the footprints all Sentinel 1A and Sentinel 1B satellites that cover ITRF sites that verify the above conditions and integrate them in a GIS project for further analyses.

![Sites, Instruments and Image footprints](img/presentation.png)

## Objectives

The objectif of the projet is to make a Python script to read the position of the ITRF instruments in the given files (see [data](data) directory), calculate the position of the ITRF sites and interrogate an API to list and download the satellite images over theses sites.

We would like as output:
* a SHP with the instrument of the processed sites ;
* a SHP with the extent of the sites ;
* a SHP with the extent of the satellite images to download.

## Data

The [data](data) are 4 textual files with :
* id of the site and id of the instrument in this site (eg: `12345S123` : `12345` + `S123`) ;
* name of the site ;
* type of the instrument ;
* name of the instrument ;
* x position ;
* y position ;
* z position ;
* the x-y-z precision.

Example (`ITRF2020_DORIS_cart.txt`):

```txt
id        name            type  code x             y             z             dx     dy     dz
10002S018 Grasse (OCA)    DORIS GR3B  4581680.3279   556166.4818  4389371.6042 0.0020 0.0025 0.0020
10002S019 Grasse (OCA)    DORIS GR4B  4581681.0445   556166.9141  4389370.9730 0.0019 0.0024 0.0017
10003S001 Toulouse        DORIS TLSA  4628047.2485   119670.6873  4372788.0168 0.0054 0.0062 0.0051
10003S003 Toulouse        DORIS TLHA  4628693.4610   119985.0770  4372104.5078 0.0034 0.0042 0.0032
10003S005 Toulouse        DORIS TLSB  4628693.6567   119985.0787  4372104.7202 0.0026 0.0039 0.0025
10077S002 Ajaccio         DORIS AJAB  4696990.0906   723981.2094  4239679.2709 1.2860 1.2898 1.2857
10202S001 Reykjavik       DORIS REYA  2585527.8355 -1044368.1434  5717159.1052 0.0148 0.0163 0.0090
```

## Project statement

1. Creation of the first SHP (instruments):
   1. Open the 4 files with [pandas](https://pandas.pydata.org/docs/reference/api/pandas.read_fwf.html).
   2. Merge it in a new DataFrame.
   3. Make a GeoDataFrame with all the instruments (convert geometry from `EPSG:4978` to `EPSG:4326`).
   4. Calculate the `site_id` and the `instrument_id` (add new columns).
   5. Remove useless columns.
   6. Save the data!
1. Creation of the second SHP (sites):
   1. Keep only the instruments that belongs to a site (look at the five first numbers of the DOMES (id) number) which hosts at least 3 instruments from 2 different measurement techniques (GNSS, DORIS, SLR or VLBI).
   2. Make a [spatial groupby (dissolve)](https://geopandas.org/en/stable/docs/user_guide/aggregation_with_dissolve.html) two join all the points from a same site.
   3. Calculate a polygon from the list of points (you will need the shapely [Polygon](https://shapely.readthedocs.io/en/latest/reference/shapely.Polygon.html#shapely.Polygon) function and the shapely [convex_hull](https://shapely.readthedocs.io/en/latest/reference/shapely.MultiPoint.html#shapely.MultiPoint.convex_hull) property).
   4. Save the data!
2. Creation of the last SHP (images):
   1. For each site, list the images (between 2022/01/01 and 2022/09/30) that are covering the extent of the site. This is very long!!! Write the information in a `<site_id>.json` temporary file to be able to restart the script if it fails.
    2. Merge all information in one GeoDataFrame.
    3. Save the data!


## API Access

You do not need to register to use the listing API.

The documentation of the API is here : https://documentation.dataspace.copernicus.eu/APIs/OpenSearch.html

We want to select images according to a **geometry**, a **start date**, an **end data** and a **collection** (`SENTINEL-2`).

Here's an example of a function you can start with:

```py
import requests

def request_images(wkt_geometry):
    items = [] # Empty list to store return elements
    # Request
    r = requests.get(
        "https://catalogue.dataspace.copernicus.eu/resto/api/collections/Sentinel2/search.json",
        params={
            "geometry": wkt_geometry,
            "startDate": "2022-01-01T00:00:00.000Z",
            "completionDate": "2022-09-30T23:59:59.999Z",
            "cloudCover": "[0,10]",
            "maxRecords": 20,
            "page": 1,
        }
    )
    # If status_code is not 200, we have an issue
    if r.status_code == 200:
        data = r.json()
        if 'features' in data:
            items += data['features']
    return items

wkt_geometry = 'POLYGON((2.349250 48.8535,2.348703 48.85293,2.350430 48.8524,2.35091 48.8530,2.349250 48.8535))'
images = request_images(wkt_geometry)
```

Look at the [data/response_example.json](data/response_example.json) file to have a look at the API response.

You may need to do several request if there are more images than the number of rows you ask (while loop). (That is **not** required for the project.)


## Data help

Instrument SHP:
* geometry type: 2d or 3d point (X,Y,Z of the instrument) ;
* `site_id`: id of the site (first 5 chars of the full id) ;
* `name`: name of the site ;
* `id`: full id of the instrument ;
* `code`: code of the instrument ;
* `type`: type of the instrument (DORIS / GNSS / SLR / VLBI) ;

Site SHP:
* geometry type: polygon (extent around the instruments)
* `site_id`: id of the site ;
* `name`: name of the site ;

Images SHP:
* geometry type: polygone (extent of the image)
* `site_id`: id of the site ;
* `id`: id of the image ;
* `title`: title of the image ;
* `thumbnail`: url of the thumbnail image ;
* `url`: url to download the image ;
* `filename`: filename ;
* `orbitNumber`: orbit number ;
* `ron`: relative orbit number ;
* `orbit`: orbit direction (ASCENDING / DESCENDING) ;
* `platform`: platform identifier.
