![image_squidhome@2x.png](http://i.imgur.com/RIvu9.png)

# Couchbase ElasticSearch sails js adaptor

Provides easy access to couchbase and elasticsearch from Sails.js & Waterline.

This module is a Waterline/Sails adapter. Its goal is to provide a set of declarative interfaces, conventions, and best-practices for integrating with all sorts of data sources. Not just databases-- external APIs, proprietary web services, or even hardware.

### Installation

To install this adapter, run:

```sh
$ npm install sails-cbes
```
### Before start keep in mind that
* Auto create Elasticsearch index
* ElasticSearch mapping is auto imported if it is defined in the model.
* To update Elasticsearch mapping you need to delete the index
* For each model a couchbase view will be created. The views are used for getting entire collection

Model with elastic search mapping example:
```javascript
module.exports = {
    identity: 'user',
    tableName: 'userTable',
    connection: 'semantic',

    attributes: {
        firstName: 'string',
        lastName: 'string',
        email: {
            type: 'string',
            defaultsTo: 'e@test.com'
        },
        avatar: 'binary',
        title: 'string',
        phone: 'string',
        type: 'string',
        favoriteFruit: {
            defaultsTo: 'blueberry',
            type: 'string'
        },
        age: 'integer', // integer field that's not auto-incrementable
        dob: 'datetime',
        status: {
            type: 'boolean',
            defaultsTo: false
        },
        percent: 'float',
        list: 'array',
        obj: 'json',
        fullName: function () {
            return this.firstName + ' ' + this.lastName;
        }
    },

    mapping: {
        "_all": {
            "enabled": false
        },
        firstName: {
            type: 'string',
            analyzer: 'whitespace',
            fields: {
                raw: {
                    type: 'string',
                    index: 'not_analyzed'
                }
            }
        },
        lastName: {
            type: 'string',
            analyzer: 'whitespace'
        },
        email: {
            type: 'string',
            analyzer: 'standard'
        },
        avatar: {
            type: 'binary'
        },
        title: {
            type: 'string',
            analyzer: 'whitespace',
        },
        phone: {
            type: 'string',
            analyzer: 'keyword'
        },
        type: {
            type: 'string',
            analyzer: 'keyword'
        },
        favoriteFruit: {
            type: 'string',
            analyzer: 'whitespace'
        },
        age: {
            type: 'integer',
            index: 'not_analyzed'
        },
        createdAt: {
            type: 'date',
            format: 'dateOptionalTime'
        },
        updatedAt: {
            type: 'date',
            format: 'dateOptionalTime'
        },
        status: {
            type: 'boolean'
        },
        percent: {
            type: 'float'
        },
        obj: {
            type: 'object'
        }
    }
};
```

### Configuration

```javascript
{
    //couchbase
    cb: {
        host: 'localhost',
        port: 8091,
        user: 'user',
        pass: 'password',
    
        bucket: {
            name: 'bucket',
            pass: 'bucketPassword'
        }
    },
    
    //elasticsearch  
    es: {
        host: ['127.0.0.1:9200'],
        log: 'error',
        index: 'index',
        numberOfShards: 5,
        numberOfReplicas: 1
    }
},
```

### Usage

This adapter exposes the following methods:

###### `find()`

+ **Status**
  + Done
  
This method accepts Elastic Search [filtered query](http://www.elasticsearch.org/guide/en/elasticsearch/reference/current/query-dsl-filtered-query.html). Only send the filtered.filter part of the query!
```javascript
var elasticsearchFilterQuery = {
    bool: {
        must: [
            {
                term: {
                    type: 'createEach'
                }
            },
            {
                terms: {
                    firstName: ['createEach_1', 'createEach_2']
                }
            }
        ]
    }
};

Semantic.User.find()
    .where(query)
    .skip(0)
    .limit(10)
    .sort({createdAt: 'desc'})
    .exec(function(err, res){
        // do something
    });
```
If you dont set no query to the find() method, find() will use couchbase view and return the entire collection.

This is the generated Elastic Search query for the above exaple:

```javascript
query: {
    filtered: {
        query: {
            bool: {
                must: [{
                    term: {
                        _type: {
                            value: modelType
                        }
                    }
                }]
            }
        },
        filter: {
            bool: {
                must: [
                    {
                        term: {
                            type: 'createEach'
                        }
                    },
                    {
                        terms: {
                            firstName: ['createEach_1', 'createEach_2']
                        }
                    }
                ]
            }
        }
    },
    size: 10,
    from: 0,
    sort: [
        {
            createdAt: {
                order: 'desc'
            }
        }
    ]
}
```
###### `findOne()`

+ **Status**
  + Done
  
This method accepts Elastic Search [filtered query](http://www.elasticsearch.org/guide/en/elasticsearch/reference/current/query-dsl-filtered-query.html). Only send the filtered.filter part of the query!

```javascript
var elasticsearchFilterQuery = {
    bool: {
        must: [
            {
                term: {
                    type: 'findOne'
                }
            }
        ]
    }
};

Semantic.User.findOne(elasticsearchFilterQuery).exec(function(err, res){
    // do something
});
```

###### `create()`

+ **Status**
  + Done
 
```javascript
Semantic.User.create({ firstName: 'createEach_1', type: 'createEach' }, function(err, res) {
    // do something
})
```
###### `createEach()`

+ **Status**
  + Done
 
```javascript
var usersArray = [
    { firstName: 'createEach_1', type: 'createEach' },
    { firstName: 'createEach_2', type: 'createEach' }
];
Semantic.User.createEach(usersArray, function(err, res) {
    // do something
})
```

###### `update()`

+ **Status**
  + Done

This method accepts Elastic Search [filtered query](http://www.elasticsearch.org/guide/en/elasticsearch/reference/current/query-dsl-filtered-query.html). Only send the filtered.filter part of the query!

Check find() method.
```javascript
var elasticsearchFilterQuery = {
    bool: {
        must: [
            {
                term: {
                    type: 'update'
                }
            },
            {
                term: {
                    firstName: 'update_1'
                }
            }
        ]
    }
};

Semantic.User.update(elasticsearchFilterQuery, {lastName: 'updated'}).exec(function(err, res){
    // do something
});
```

###### `destroy()`

+ **Status**
  + Done

This method accepts Elastic Search [filtered query](http://www.elasticsearch.org/guide/en/elasticsearch/reference/current/query-dsl-filtered-query.html). Only send the filtered.filter part of the query!

Check find() method.

```javascript
var elasticsearchFilterQuery = {
    bool: {
        must: [
            {
                term: {
                    type: 'getRawCollection'
                }
            }
        ]
    }
};

Semantic.User.destroy(elasticsearchFilterQuery).limit(999999).exec(function(err, res){
    // do something
});
```
###### `getRawCollection()`

+ **Status**
  + Done
 
This method returns raw data from Couchbase view.

``` javascript
Semantic.User.getRawCollection(function(err, res){
    // do something
});
```

### Development

Check out **Connections** in the Sails docs, or see the `config/connections.js` file in a new Sails project for information on setting up adapters.

### Running the tests

In your adapter's directory, run:

```sh
$ npm test
```

### Questions?

See [`FAQ.md`](./FAQ.md).



### More Resources

- [Stackoverflow](http://stackoverflow.com/questions/tagged/sails.js)
- [#sailsjs on Freenode](http://webchat.freenode.net/) (IRC channel)
- [Twitter](https://twitter.com/sailsjs)
- [Professional/enterprise](https://github.com/balderdashy/sails-docs/blob/master/FAQ.md#are-there-professional-support-options)
- [Tutorials](https://github.com/balderdashy/sails-docs/blob/master/FAQ.md#where-do-i-get-help)
- <a href="http://sailsjs.org" target="_blank" title="Node.js framework for building realtime APIs."><img src="https://github-camo.global.ssl.fastly.net/9e49073459ed4e0e2687b80eaf515d87b0da4a6b/687474703a2f2f62616c64657264617368792e6769746875622e696f2f7361696c732f696d616765732f6c6f676f2e706e67" width=60 alt="Sails.js logo (small)"/></a>


### License

**[MIT](./LICENSE)**
&copy; 2015 [aronluigi/Kreditech](https://github.com/aronluigi) & [contributors]
[Mohammad Bagheri](https://github.com/bagheri-m1986), [Robert Savu](https://github.com/r-savu), [Tiago Amorim](https://github.com/tiagoamorim85) & contributors

[Sails](http://sailsjs.org) is free and open-source under the [MIT License](http://sails.mit-license.org/).


[![githalytics.com alpha](https://cruel-carlota.pagodabox.com/8acf2fc2ca0aca8a3018e355ad776ed7 "githalytics.com")](http://githalytics.com/balderdashy/waterline-test/README.md)


