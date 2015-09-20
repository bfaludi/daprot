# daprot

You can iterate over a dataset and load your data into the right place while it will convert and transform the data as well. At the end you will have a list of dictionaries containing your fixed data.

I know It's complicated but If we see some examples It will be easier to understand.

However, I suggest you to check the following link as well.

- [dm](https://github.com/bfaludi/dm) Read about how the `route` attribute works.
- [funcomp](https://github.com/bfaludi/funcomp) See all possibility to the `transforms` property.

... and some nice extensions for this package:

- [uniopen](https://github.com/bfaludi/uniopen) Open any kind of file/url/database with the same interface.
- [forcedtypes](https://github.com/bfaludi/forcedtypes) To force crappy quality data into python type. Less error, less line. :)

The package is compatible with Python 2 & 3.

## Quick tutorial

### Grab foursquare venues

Let's see an example where we're using foursquare's [venue search API](https://developer.foursquare.com/docs/explore#req=venues/search%3Fll%3D40.7,-74).

We can write a short Python code and iterate over the response. During the process we are able to map the fields into columns and convert (or transform) them.

```python
import dm
import json
import daprot as dp
import uniopen # You can use `requests` and `urllib` package too.
import forcedtypes as t # Using this package because I want to force `postal_code` to integer.

foursquare_venue_search_url = 'https://api.foursquare.com/v2/venues/search?ll=40.7,-74&oauth_token=...'

class Venues(dp.SchemaFlow):
    id = dp.Field()
    name = dp.Field()
    phone_number = dp.Field('contact/phone')
    twitter_id = dp.Field('contact/twitter')
    facebook_id = dp.Field('contact/facebook')
    address = dp.Field('location/address')
    postal_code = dp.Field('location/postalCode', type=t.Int)
    city = dp.Field('location/city')
    country = dp.Field('location/country')
    latitude = dp.Field('location/lat', type=float)
    longitude = dp.Field('location/lng', type=float)
    categories = dp.Field('categories/*/name')
    check_in_count = dp.Field('stats/checkinsCount', type=int)
    tip_count = dp.Field('stats/tipCount', type=int)

with uniopen.Open(foursquare_venue_search_url) as rs:
    api_result = dm.Mapper(json.load(rs))
    for venue in Venues(api_result['response/venues/!'], mapper=dp.mapper.NAME):
        print(venue)
```

This is the output of the first line.

```python
{'phone_number': u'+12128033822', 'city': u'Brooklyn', 'name': u'Brooklyn Bridge Park', 'check_in_count': 37081, 'country': u'United States', 'facebook_id': u'104475634308', 'twitter_id': u'nycparks', 'longitude': -73.9965033531189, 'postal_code': 11201L, 'tip_count': 224, 'address': u'Main St', 'latitude': 40.70227697066692, 'id': u'430d0a00f964a5203e271fe3', 'categories': [u'Park']}
```
... and it was the original result of the same venue:

This is a single venue in JSON.

```json
{  
   "id":"430d0a00f964a5203e271fe3",
   "name":"Brooklyn Bridge Park",
   "contact":{  
      "phone":"+12128033822",
      "formattedPhone":"+1 212-803-3822",
      "twitter":"nycparks",
      "facebook":"104475634308",
      "facebookUsername":"BartowPell",
      "facebookName":"Bartow-Pell Mansion Museum"
   },
   "location":{  
      "address":"Main St",
      "crossStreet":"Plymouth St",
      "lat":40.70227697066692,
      "lng":-73.9965033531189,
      "distance":389,
      "postalCode":"11201",
      "cc":"US",
      "city":"Brooklyn",
      "state":"NY",
      "country":"United States",
      "formattedAddress":[  
         "Main St (Plymouth St)",
         "Brooklyn, NY 11201",
         "United States"
      ]
   },
   "categories":[  
      {  
         "id":"4bf58dd8d48988d163941735",
         "name":"Park",
         "pluralName":"Parks",
         "shortName":"Park",
         "icon":{  
            "prefix":"https://ss3.4sqi.net/img/categories_v2/parks_outdoors/park_",
            "suffix":".png"
         },
         "primary":true
      }
   ],
   "verified":true,
   "stats":{  
      "checkinsCount":37083,
      "usersCount":22618,
      "tipCount":224
   },
   "url":"http://nyc.gov/parks",
   "specials":{  
      "count":0,
      "items":[  

      ]
   },
   "hereNow":{  
      "count":7,
      "summary":"7 people are here",
      "groups":[  
         {  
            "type":"others",
            "name":"Other people here",
            "count":7,
            "items":[  

            ]
         }
      ]
   },
   "referralId":"v-1442770357"
}
```

## License

Copyright © 2015 Bence Faludi.

Distributed under the GPLv3 License.

