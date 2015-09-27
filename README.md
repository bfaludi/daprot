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

### 1. Grab foursquare venues

Let's see an example where we're using foursquare's [venue search API](https://developer.foursquare.com/docs/explore#req=venues/search%3Fll%3D40.7,-74).

We can write a short Python code and iterate over the response. During the process we are able to map the fields into columns and convert (or transform) them.

```python
import dm
import json
import daprot as dp
import uniopen # You can use `requests` or `urllib` package too.
import forcedtypes as t # Using this package because I want to force `postal_code` value to integer.

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
{
  'phone_number': u'+12128033822',
  'city': u'Brooklyn', 
  'name': u'Brooklyn Bridge Park', 
  'check_in_count': 37081, 
  'country': u'United States', 
  'facebook_id': u'104475634308', 
  'twitter_id': u'nycparks', 
  'longitude': -73.9965033531189, 
  'postal_code': 11201L, 
  'tip_count': 224, 
  'address': u'Main St', 
  'latitude': 40.70227697066692, 
  'id': u'430d0a00f964a5203e271fe3', 
  'categories': [u'Park']
}
```
... and it was the original result of the same venue:

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

### 2. Read and convert values

We have a fixed width text file and It contains product informations. (e.g.: id, name, formatted price in HUF, quantity, last updated date).

```txt
P0001   Tomato           449      1     2015.09.20 20:00
P0002   Paprika          399      1     2015.09.20 20:02
P0003      Cucumber      199      10
P0004   Chicken nuggets  999,5    1     2015.09.20 12:47
P0005   Chardonnay       2 399    1     2015.09.20 07:31
```

We can write a short Python script to read this file and convert `price` information to float, remove extra whitespaces.

```python
import uniopen
import datetime
import daprot as dp
import forcedtypes as t

class Groceries(dp.SchemaFlow):
    id = dp.Field('0:8')
    name = dp.Field('8:25', type=str, transforms=str.strip)
    price = dp.Field('25:34', type=t.new(t.Float, locale='hu_HU'))
    quantity = dp.Field('34:40', type=int)
    updated_at = dp.Field('40:', type=t.Datetime, default_value=datetime.datetime.now)

with uniopen.Open('groceries.txt', 'r', encoding='utf-8') as rs:
    print(list(Groceries(rs)))
```

This is the result:

```python
[{
  'price': 449.0,
  'quantity': 1,
  'id': u 'P0001',
  'name': 'Tomato',
  'updated_at': datetime.datetime(2015, 9, 20, 20, 0)
}, {
  'price': 399.0,
  'quantity': 1,
  'id': u 'P0002',
  'name': 'Paprika',
  'updated_at': datetime.datetime(2015, 9, 20, 20, 2)
}, {
  'price': 199.0,
  'quantity': 10,
  'id': u 'P0003',
  'name': 'Cucumber',
  'updated_at': datetime.datetime(2015, 9, 20, 20, 17, 28, 846543)
}, {
  'price': 999.5,
  'quantity': 1,
  'id': u 'P0004',
  'name': 'Chicken nuggets',
  'updated_at': datetime.datetime(2015, 9, 20, 12, 47)
}, {
  'price': 2399.0,
  'quantity': 1,
  'id': u 'P0005',
  'name': 'Chardonnay',
  'updated_at': datetime.datetime(2015, 9, 20, 7, 31)
}]
```

### 3. Create nested elements on demand.

You can create sub-lists and sub-dictionaries during the mapping. Let's see a new example where we're using the same foursquare's [API](https://developer.foursquare.com/docs/explore#req=venues/search%3Fll%3D40.7,-74).

```python
import dm
import json
import daprot as dp
import uniopen # You can use `requests` or `urllib` package too.
import forcedtypes as t # Using this package because I want to force `postal_code` value to integer.

foursquare_venue_search_url = 'https://api.foursquare.com/v2/venues/search?ll=40.7,-74&oauth_token=...'

class Contact(dp.SchemaFlow):
    phone_number = dp.Field('contact/phone')
    twitter_id = dp.Field('contact/twitter')
    facebook_id = dp.Field('contact/facebook')
    address = dp.Field('location/address')
    postal_code = dp.Field('location/postalCode', type=t.Int)
    city = dp.Field('location/city')
    country = dp.Field('location/country')
    latitude = dp.Field('location/lat', type=float)
    longitude = dp.Field('location/lng', type=float)

class Category(dp.SchemaFlow):
    category_id = dp.Field('id')
    name = dp.Field()

class Venues(dp.SchemaFlow):
    venue_id = dp.Field('id')
    name = dp.Field()
    categories = dp.ListOf(Category, route='categories')
    contact = dp.DictOf(Contact) # Using the same `route` as the parent.
    check_in_count = dp.Field('stats/checkinsCount', type=int)
    tip_count = dp.Field('stats/tipCount', type=int)

with uniopen.Open(foursquare_venue_search_url) as rs:
    api_result = dm.Mapper(json.load(rs))
    for venue in Venues(api_result['response/venues/!'], mapper=dp.mapper.NAME):
        print(venue)
```

As you can see You can define `DictOf` and `ListOf` items easily. Do not forget, the `transforms` and `default_value` attributes are still operational.

## License

Copyright © 2015 Bence Faludi.

Distributed under the GPLv3 License.

