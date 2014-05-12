# mapmyfitness-python

[![Build Status](https://api.travis-ci.org/JasonSanford/mapmyfitness-python.png?branch=master)](https://travis-ci.org/JasonSanford/mapmyfitness-python)&nbsp;[![Coverage Status](https://coveralls.io/repos/JasonSanford/mapmyfitness-python/badge.png?branch=master)](https://coveralls.io/r/JasonSanford/mapmyfitness-python?branch=master)

A Python wrapper for the [MapMyFitness API](https://developer.mapmyapi.com/)

## Status

This is a work in progress. Current endpoints implemented are:

* [Routes](#routes)
* [Workouts](#workouts)
* [Users](#users)
* [Activity Types](#activity-types)

## Contributing

See [Contributing](CONTRIBUTING.md)

## Installation

Install via pip:

    pip install mapmyfitness

or from local sources:

    python setup.py install

## Dependencies

* Python 2.7, 3.2 or 3.3
* [Requests](http://docs.python-requests.org/en/latest/) takes care of our HTTP chatting, and is automatically installed when using the steps above.

## Usage

Instantiate an instance of `MapMyFitness` with your API key and an access token representing a user:

```python
from mapmyfitness import MapMyFitness
mmf = MapMyFitness(api_key='not-so-secret', access_token='super-secret')
```

Optionally pass `True` for `cache_finds`. This will cache instances of objects fetched through the `find` method. This is helpful for objects that aren't likely to change, like [Activity Types](#activity-type).

```python
from mapmyfitness import MapMyFitness
mmf = MapMyFitness(api_key='not-so-secret', access_token='super-secret', cache_finds=True)
```

### Routes

Implements behaviors: [find](#find), [search](#search), [create](#create), [delete](#delete), [update](#update)

#### Search Parameters

* `user` - integer - A user id to find routes for
* `users` - list or tuple - a collection of user ids to find routes for
* `close_to_location` - list or tuple - A 2-list or 2-tuple containing a latitude and longitude to search for routes near
* `minimum_distance` - int or float - The minimum distance, in meters, of routes to search for - only for use with `close_to_location`
* `maximum_distance` - int or float - The maximum distance, in meters, of routes to search for - only for use with `close_to_location`

\* One of `user`, `users` or `close_to_location` parameters must be passed.

Search for routes created by a user:

```python
routes_paginator = mmf.route.search(user=9118466)  # returns a paginator object
```

Get route geometry:

    >>> route.points()
    [
        {'lat': 39.74942025, 'lng': -104.99598683, 'ele': 1604.96},
        {'lat': 39.74954872, 'lng': -104.99627271, 'ele': 1605.48},
        ...
    ]

Prefer GeoJSON?:

    >>> route.points(geojson=True)
    {
        'type': 'LineString',
        'coordinates': [
            (-104.9959868, 39.74942025),
            (-104.9961131, 39.74958007),
            ...
        ]
    }

### Workouts

Implements behaviors: [find](#find), [search](#search), [create](#create), [delete](#delete), [update](#update)

### Users

Implements behaviors: [find](#find), [search](#search)

### Activity Types

Implements behaviors: [find](#find)

## Behaviors

### find

Find a single object by its unique id. Returns a object or raises `mapmyfitness.exceptions.NotFoundException` if no object is found.

Example:

```python
route = mmf.route.find(348949363)
print(route.name)  # '4 Mile Lunch Run'
```

### search

Search for objects. Returns a [Paginator object](https://github.com/JasonSanford/mapmyfitness-python/blob/master/mapmyfitness/paginator.py#L8) to easily page through large result sets.

See [this gist](https://gist.github.com/JasonSanford/ddbe06832e5d17061b8a) for implementation details.

#### Paginator Properties

* `count` - returns the total number of objects on all pages.
* `num_pages` - returns the number of pagees in the paginator.
* `page_range` - returns a range of page numbers to allow for iteration of pages.

#### Paginator Methods

* `page(<page_number>)` - returns a [Page object](https://github.com/JasonSanford/mapmyfitness-python/blob/master/mapmyfitness/paginator.py#L63) containing objects on a specific page.

```python
start_datetime = datetime.datetime(2014, 1, 1)
end_datetime = datetime.datetime(2015, 1, 1)

workouts_paginator = mmf.workout.search(user=9118466, per_page=40, started_after=start_datetime, started_before=end_datetime)

page_count = workouts_paginator.num_pages  # 2
page_range = workouts_paginator.page_range # [1, 2]
total_count = workouts_paginator.count # 58

for page_num in page_range:
    the_page = workouts_paginator.page(page_num)
    print(the_page)  # <Page 1 of 2>
    for workout in the_page:
        print(workout.start_datetime)  # 2014-01-02 02:59:53+00:00
```

### create

Create an object.

### delete

Delete an object.

Example:

```python
mmf.route.delete(348949363)
```

Returns `None` on success or raises `mapmyfitness.exceptions.NotFoundException` if the object doesn't exist.

### update

Update an object.
