![# Welcome to my adventure](/images/Sat3.jpeg)

# Satellite Imagery Analysis with Python


### Setup 
  - Planet’s Python Client [API](https://pypi.org/project/planet/)
  - before installing Rasterio make sure [numpy](http://www.numpy.org) is already installed
  - [Rasterio](https://github.com/mapbox/rasterio): for organizing and storing gridded raster datasets as satellite imagery (GeoTIFF, Numpy N-dimensional arrays, GeoJSON.numpy)
  - [matplotlib](https://matplotlib.org)
  - [requests](http://docs.python-requests.org/en/master/) (you might not need to if you already have miniconda installed: '$ pip install requests Requirement already satisfied: requests in /miniconda3/envs/...' )

#### Data Extraction
If you are a planeteer then you`ll get your data from: [Planet Explorer](https://www.planet.com/products/explorer/), of course, the 14 days trial. 

Now, time to get the data: and I am proud to say that, at this moment in time, this is the most complete version that you will find online:

Get to your Jupyter Notebook and start typing: 


In 1:  from planet import api
       client = api.ClientV1()

Get your coordinates with this fast hack: at geojson.io and pick your area of interest and copy paste your coordinates

In 2:   # Silicon Valley, San Francisco, CA, USA, coordinates
        geojson_geometry = {
          "type": "Polygon",
          "coordinates": [
                  [
                    [
                      -122.43576049804688,
                      37.51299386065851
                    ],
                    [
                      -122.34786987304686,
                      37.155938651244625
                    ],
                    [
                      -122.07595825195312,
                      36.948794297566366
                    ],
                    [
                      -121.8878173828125,
                      37.18110808791507
                    ],
                    [
                      -122.43576049804688,
                      37.51299386065851
                    ]
                  ]
                ]
              }

In 3:   # get images that overlap with our area of interest 
        geometry_filter = {
          "type": "GeometryFilter",
          "field_name": "geometry",
          "config": geojson_geometry
        }

        # get images acquired within a date range
        date_range_filter = {
          "type": "DateRangeFilter",
          "field_name": "acquired",
          "config": {
            "gte": "2018-08-31T00:00:00.000Z",
            "lte": "2018-09-01T00:00:00.000Z"
          }
        }

        # only get images which have <50% cloud coverage
        cloud_cover_filter = {
          "type": "RangeFilter",
          "field_name": "cloud_cover",
          "config": {
            "lte": 0.5
          }
        }

        # combine our geo, date, cloud filters
        combined_filter = {
          "type": "AndFilter",
          "config": [geometry_filter, date_range_filter, cloud_cover_filter]
        }

Make sure you`ve got your API key from [Planet's Account](https://www.planet.com/account/#/). 

In 4:   import os
        import json
        import requests
        from requests.auth import HTTPBasicAuth

        os.environ['PL_API_KEY']='HERE YOU INPUT YOU API Key'

        # API Key stored as an env variable
        PLANET_API_KEY = os.getenv('PL_API_KEY') 
        
        # will get a 4 band image with spectral data for Red, Green, Blue and Near-infrared values
        item_type = "PSScene4Band"

        # API request object
        search_request = {
          "interval": "day",
          "item_types": [item_type], 
          "filter": combined_filter
        }

        # fire off the POST request
        search_result = \
          requests.post(
            'https://api.planet.com/data/v1/quick-search',
            auth=HTTPBasicAuth(PLANET_API_KEY, ''),
            json=search_request)

        print(json.dumps(search_result.json(), indent=1))




To download the image, we need to activate it. Once the activation status becomes “active,” we can then download the image of interest.

Exploring the Satellite Imagery
The python’s Rasterio library makes it very easy to explore satellite images. Satellite Images are nothing but grids of pixel-values and hence can be interpreted as multidimensional arrays.

- figure out how to include a notebook doc - that can be scrll-able. 


