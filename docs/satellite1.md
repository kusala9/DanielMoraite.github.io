![# Welcome to my adventure](/images/sat3.jpeg)

# Satellite Imagery Analysis with Python


## PART I

### Setup 
  - Planet’s Python Client [API](https://pypi.org/project/planet/)
  - before installing Rasterio make sure [numpy](http://www.numpy.org) is already installed
  - [Rasterio](https://github.com/mapbox/rasterio): for organizing and storing gridded raster datasets as satellite imagery (GeoTIFF, Numpy N-dimensional arrays, GeoJSON.numpy)
  - [matplotlib](https://matplotlib.org)
  - [requests](http://docs.python-requests.org/en/master/) (you might not need to if you already have miniconda installed: '$ pip install requests Requirement already satisfied: requests in /miniconda3/envs/...' )

#### Data Extraction
If you are a planeteer then you'll get your data from: [Planet Explorer](https://www.planet.com/products/explorer/), of course, the 14 days trial. 

Now, time to get the data: and I am proud to say that, at this moment in time, this is the most complete version that you will find online:

Get to your Jupyter Notebook and start typing: 


       from planet import api
       client = api.ClientV1()

      # Get your coordinates with this fast hack: at geojson.io and pick your area of interest and copy paste your coordinates bellow:
      # I have picked some Silicon Valley, San Francisco, CA, USA, coordinates:

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


You might want to make sure you get the right data: setting up the time frame, if clouds, by adding some filters:

        # get images that overlap with our area of interest 
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

Make sure you've got your API key from [Planet's Account](https://www.planet.com/account/#/). 

        import os
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


To download a photo you need to pick one and set the asset type: 'analytic' or 'analytic_dn'(which is Planet old format), as well the activation status needs to be “active”. Just have patience, it might take a few minutes (3 to 5 min). 

    # extract image IDs only
    image_ids = [feature['id'] for feature in search_result.json()['features']]
    print(image_ids)

    # For demo purposes, just grab the first image ID
    id0 = image_ids[0]
    id0_url = 'https://api.planet.com/data/v1/item-types/{}/items/{}/assets'.format(item_type, id0)

    # Returns JSON metadata for assets in this ID. Learn more: planet.com/docs/reference/data-api/items-assets/#asset
    result = \
      requests.get(
        id0_url,
        auth=HTTPBasicAuth(PLANET_API_KEY, '')
      )

    # List of asset types available for this particular satellite image
    print(result.json().keys())

    # This is "inactive" if the "analytic" asset has not yet been activated; otherwise 'active'
    print(result.json()['analytic_dn']['status'])

Then activate it: 

    # activate the asset for download:
    links = result.json()[u"analytic_dn"]["_links"]
    self_link = links["_self"]
    activation_link = links["activate"]

    # Request activation of the 'analytic' asset:
    activate_result = \
      requests.get(
        activation_link,
        auth=HTTPBasicAuth(PLANET_API_KEY, '')
      )

And while saying 'Activating' you can probe if ready with: 

    activation_status_result = \
      requests.get(
        self_link,
        auth=HTTPBasicAuth(PLANET_API_KEY, '')
      )

    print(activation_status_result.json()["status"])

When 'Active' just download it:

    # Image can be downloaded by making a GET with your Planet API key, from here:
    download_link = activation_status_result.json()["location"]
    print(download_link)

And check you download folder. If you don't have the right app to view a .tiff then don't get alarmed if the image looks blank in your regular image viewer. And make sure you have enough space on disc. 
_________________________

Please find the entire code [here](https://github.com/DanielMoraite/DanielMoraite.github.io/blob/master/assets/Downloading%20from%20Planet-Copy1.ipynb)
All you'll have to do is pick you own coordinates, instead of spending hour figuring the above. 

> Active data will be only from California, USA. You might find other active data (like for example Romania, though it will be available for areas bigger than 10K Square Kilometres which will turn a error message for exceeding your monthly download limit.

_________________________
##### Data Trouble Shooting (hopefully you won't need it):

- [Download quota for Open California dataset?](https://gis.stackexchange.com/questions/238803/download-quota-for-open-california-dataset)
- [Why am I unable to activate certain Planet Labs images through the Python API?](https://gis.stackexchange.com/questions/217716/why-am-i-unable-to-activate-certain-planet-labs-images-through-the-python-api/217787) in case you are picking areas from the allowed region of California and still get the innactive feedback. 

HAVE FUN!

-------------------------


# Exploring the Satellite Imagery: 

## PART II

Time to use python’s Rasterio library since satellite images are grids of pixel-values and can be interpreted as multidimensional arrays.

    import math
    import rasterio
    import matplotlib.pyplot as plt

    image_file = "image.tif"  # just add the image downloaded from Planet.
    sat_data = rasterio.open(image_file)

Image dimmension in metres as well rows and columns: 

    width_in_projected_units = sat_data.bounds.right - sat_data.bounds.left
    height_in_projected_units = sat_data.bounds.top - sat_data.bounds.bottom

    print("Width: {}, Height: {}".format(width_in_projected_units, height_in_projected_units))

    print("Rows: {}, Columns: {}".format(sat_data.height, sat_data.width))

Pixel conversion to latitude and longitude: 

    # Upper left pixel
    row_min = 0
    col_min = 0

    # Lower right pixel.  Rows and columns are zero indexing.
    row_max = sat_data.height - 1
    col_max = sat_data.width - 1

    # Transform coordinates with the dataset's affine transformation.
    topleft = sat_data.transform * (row_min, col_min)
    botright = sat_data.transform * (row_max, col_max)

    print("Top left corner coordinates: {}".format(topleft))
    print("Bottom right corner coordinates: {}".format(botright))

Storing the bands(B,G,R,N infrared) in numpy array: 

    print(sat_data.count)

    # sequence of band indexes
    print(sat_data.indexes)

### Visualising the Satellite Imagery

    # Load the 4 bands into 2d arrays - recall that we previously learned PlanetScope band order is BGRN.
    b, g, r, n = sat_data.read()

    # Displaying the blue band.

    fig = plt.imshow(b)
    plt.show()

![# Welcome to my adventure](/images/Cali1.jpeg)

    # Displaying the green band.

    fig = plt.imshow(g)
    fig.set_cmap('gist_earth')
    plt.show()

![# Welcome to my adventure](/images/Cali2.jpeg)

    # Displaying the red band.

    fig = plt.imshow(r)
    fig.set_cmap('inferno')
    plt.colorbar()
    plt.show()

![# Welcome to my adventure](/images/Cali3.jpeg)

    # Displaying the infrared band.

    fig = plt.imshow(n)
    fig.set_cmap('winter')
    plt.colorbar()
    plt.show()

![# Welcome to my adventure](/images/Cali31.jpeg)

This can be a very useful practice for data preparation for machine & deep learning approached doan the road, including the NDVI which can add to object classification for vegetation (trees, parks, etc.).

Hope you are enjoying it! 

----------------------
----------------------



