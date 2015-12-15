# hapi-pagination

[![NPM Version](https://img.shields.io/npm/v/hapi-pagination.svg)](https://npmjs.org/package/hapi-pagination)
[![Build Status](https://travis-ci.org/fknop/hapi-pagination.svg)](https://travis-ci.org/fknop/hapi-pagination)
[![Coverage Status](https://coveralls.io/repos/fknop/hapi-pagination/badge.svg?branch=master&service=github)](https://coveralls.io/github/fknop/hapi-pagination?branch=master)
[![Dependency Status](https://david-dm.org/fknop/hapi-pagination.svg)](https://david-dm.org/fknop/hapi-pagination)
[![bitHound Overalll Score](https://www.bithound.io/github/fknop/hapi-pagination/badges/score.svg)](https://www.bithound.io/github/fknop/hapi-pagination)

Hapi plugin to handle 'custom' resources pagination in json only.
Support only get method for now.

## How to install

```
npm install hapi-pagination --save
```

## How to use

hapi-pagination uses EcmaScript 6. The following examples will use it as well.

The plugin works with settings that you can override. You can override the
default value of a setting and even the default name. It allows you to customize
your calls to your API to suit your needs.

### Options

See the default options object below.

#### The query parameters

The plugin accepts query parameters to handle the pagination, you can customize
these parameters with the following options:

* `limit`: The number of resources by page. Default value is 25, default name is
  limit.
* `page`: The number of the page that will be returned. Default value is 1,
  default name is page.
* `pagination`: Allows you to enable, disable pagination for one request. Default
  value is true (enabled), default name is pagination.
* `invalid`: This is `NOT` a query parameter, but it allows you to customize the
  behavior if the validation of limit and page fails. By default, it sets the
  defaults, you can set it to 'badRequest' that will send you a `400 - Bad
  Request`.

Notes:
* You can access to limit, page and pagination in the handler method through `request.query`.
* If the pagination is set to false, the metadata object will not be a part of
  the response but the `pagination` parameter will still be accessible through
  `request.query`

#### The metadata

The plugin will generate a metadata object alongside your resources, you can
customize this object with the following options:


* `name`: The name of the metadata object. Default is 'meta'.
* `baseUri`: The base uri for the generated links. Default is ''.
* `count`: The number of rows returned. Default name is count. Enabled by default.
* `totalCount`: The total numbers of rows available. Default name is totalCount.
  Enabled by default.
* `pageCount`: The total number of pages available. Default name is pageCount,
  enabled by default.
* `self`: The link to the requested page. Default name is self, enabled by
  default.
* `previous`: The link to the previous page. Default name is previous, enabled by
  default. null if no previous page is available.
* `next`: Same than previous but with next page.
* `first`: Same than previous but with first page.
* `last`: Same than previous but with last page.
* `page`: The page number requested. Default name is page, disabled by default.
* `limit`: The limit requested. Default name is limit, disabled by default.

#### The results

* `name`: the name of the results array, results by default.
* `reply`: Object with:
    + `paginate`: The name of the paginate method (see below), paginate by
	  default.

#### The routes

* `include`: An array of routes that you want to include, support \*.
  Default to '\*'.
* `exclude`: An array of routes that you want to exclude. Useful when include is
  '\*'. Default to empty array.
* `override`: An array ob object containing: (default to empty array)
    + `routes`: An array of routes for which you want to override the defaults.
    + `limit`: The overrided limit.
    + `page`: The overrided page.

#### reply.paginate(Array, [totalCount])

The method is an helper method. This is a shortcut for:

```javascript
reply({results: results, totalCount: totalCount});
```

You can also reply the array and set the totalCount by adding the totalCount
(with whatever name you chose) to the request object.

```
request.totalCount = 10;
reply(results);
```

##### WARNING: If the results is not an array, the program will throw an implementation error.

If totalCount is not exposed through the request object
or the reply.paginate method, the following attributes will be
set to null if they are active.
 * `last`
 * `pageCount`
 * `totalCount`
 * `next`

You can still have those four attributes by exposing totalCount even if
totalCount is set to false.

#### The defaults options

```javascript
const options = {
    query: {
        page: {
            name: 'page',
            default: 1
        },
        limit: {
            name: 'limit',
            default: 25
        },
        pagination: {
            name: 'pagination',
			default: true
		}
        invalid: 'defaults'
    },

    meta: {
        name: 'meta',
        count: {
            active: true,
            name: 'count'
        },
        totalCount: {
            active: true,
            name: 'totalCount'
        },
        pageCount: {
            active: true,
            name: 'pageCount'
        },
        self: {
            active: true,
            name: 'self'
        },
        previous: {
            active: true,
            name: 'previous'
        },
        next: {
            active: true,
            name: 'next'
        },
        first: {
            active: true,
            name: 'first'
        },
        last: {
            active: true,
            name: 'last'
        },
        page: {
            active: false,
            // name == default.query.page.name
        },
        limit: {
            active: false
            // name == default.query.limit.name
        }
    },

    results: {
		name: 'results'
    },
	reply: {
        paginate: 'paginate'
	},

    routes: {
        include: ['*'],
        exclude: [],

        override: [{
            routes: [],
            limit: 25,
            page: 1
        }]
    }
};
```


### Simple example

```javascript
const Hapi = require('hapi');

let server = new Hapi.Server();

// Add your connection

server.register(require('hapi-pagination'), (err) => {
    if (err)
        throw err;
});
```

### Example with options

```javascript
const Hapi = require('hapi');

let server = new Hapi.Server();

// Add your connection

const options = {
    query: {
      page: {
        name: 'the_page' // The page parameter will now be called the_page
      },
      limit: {
        name: 'per_page', // The limit will now be called per_page
        default: 10       // The default value will be 10
      }
    },
     meta: {
        name: 'metadata', // The meta object will be called metadata
        count: {
            active: true,
            name: 'count'
        },
        pageCount: {
            name: 'totalPages'
        },
        self: {
            active: false // Will not generate the self link
        },
        first: {
            active: false // Will not generate the first link
        },
        last: {
            active: false // Will not generate the last link
        }
     },
     routes: {
         include: ['/users', '/accounts', '/persons', '/'],

         // Overrides default values of specified routes.
         // Must be specified in include (or *)
         override: [{
             // Overrides default values for routes '/accounts' and '/persons'
             routes: ['/accounts', '/persons'],
             limit: 25,
             page: 1
         }, { // Overrides default values for route '/'
            routes: ['/'],
            limit: 100,
            page: 1
         }]
     }
};

server.register({register: require('hapi-pagination'), options: options}, (err)
=> {
    if (err)
        throw err;
});
```

## Tests

Make sure you have `lab` and `code` installed and run :

```
npm test
```

## Contribute

Post an issue if you encounter a bug or an error in the documentation.
