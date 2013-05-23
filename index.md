---
published: true
layout: index

---

---
layout: index
---

# ReliefWeb API - Documentation

This the documentation for the ReliefWeb API.

The API allows to fetch content from [ReliefWeb](http://reliefweb.int) like the latest headlines, job offers or disasters.

- [versions](#Versions)
- [entities](#Entities)
- [methods](#Methods)


## Versions

The current available version is beta version and is labeled **`v0`**.

By calling the API with just the version number, you can discover the available API endpoints:

```
curl -XGET 'http://api.rwdev.org/v0'
```

```json
{
	"version": 0,
	"status": 200,
	"data": {
		"title": "ReliefWeb API",
		"endpoints": {
			"report": {
				"info": "http:\/\/api.rwdev.org\/v0\/report\/info",
				"list": "http:\/\/api.rwdev.org\/v0\/report\/list",
				"item": "http:\/\/api.rwdev.org\/v0\/report\/[ID]"
			},
			"job": {
				"info": "http:\/\/api.rwdev.org\/v0\/job\/info",
				"list": "http:\/\/api.rwdev.org\/v0\/job\/list",
				"item": "http:\/\/api.rwdev.org\/v0\/job\/[ID]"
			},
			"training": {
				"info": "http:\/\/api.rwdev.org\/v0\/training\/info",
				"list": "http:\/\/api.rwdev.org\/v0\/training\/list",
				"item": "http:\/\/api.rwdev.org\/v0\/training\/[ID]"
			},
			"country": {
				"info": "http:\/\/api.rwdev.org\/v0\/country\/info",
				"list": "http:\/\/api.rwdev.org\/v0\/country\/list",
				"item": "http:\/\/api.rwdev.org\/v0\/country\/[ID]"
			},
			"disaster": {
				"info": "http:\/\/api.rwdev.org\/v0\/disaster\/info",
				"list": "http:\/\/api.rwdev.org\/v0\/disaster\/list",
				"item": "http:\/\/api.rwdev.org\/v0\/disaster\/[ID]"
			}
		}
	}
}
```


## Entities

Entities correspond to the main content on [ReliefWeb](http://reliefweb.int).

Currently data can be fetched for 5 entities:

- **Report**:

  This entity type corresponds to updates, headlines or maps available on [ReliefWeb - Updates](http://reliefweb.int/updates).

- **Job**:

  This entity type corresponds to the job offers available on [ReliefWeb - Jobs](http://reliefweb.int/jobs).

- **Training**:

  This entity type corresponds to the training opportunities available on [ReliefWeb - Training](http://reliefweb.int/training).

- **Country**:

  This entity type corresponds to the countries available on [ReliefWeb - Country](http://reliefweb.int/countries).

- **Disaster**:

  This entity type corresponds to the disasters covered by [ReliefWeb - Disaster](http://reliefweb.int/disasters).


## Methods

Methods are the way to get the data for an entity type.

They follow the same pattern:

```
http://[APIURL]/[VERSION]/[ENTITY]/[METHOD|ITEMID]
```

Currently 3 methods are defined.

1. `info`
2. `list`
3. `[itemid]`

The last one is not a method per se, but follows a similar pattern and allows to get the data for a particular entity item.

> **Only the HTTP method GET is allowed.**


### 1. info

This method returns information about the entity, mainly fields definition.

> **This method doesn't accept parameters.**

Example:

```
curl -XGET 'http://api.rwdev.org/v0/country/info'
```

```json
{
	"version": 0,
	"satus": 200,
	"data": {
		"type": "country",
		"info": {
			"fields": {
				"id": {
					"type": "number",
					"sortable": true
				},
				"name": {
					"type": "string",
					"exact": true,
					"sortable": true
				},
				"shortname": {
					"type": "string",
					"exact": true,
					"sortable": true
				},
				"iso3": {
					"type": "string",
					"exact": true,
					"sortable": true
				},
				"current": {
					"type": "boolean"
				},
				"featured": {
					"type": "boolean"
				},
				"description": {
					"type": "string"
				}
			}
		}
	}
}
```

The result is a list of properties for each field.

| Field property     | description | values |
| ------------------ | ----------- | ------ |
| **type**           | indicates what kind of value the field can contain | `boolean`, `date`, `number` or `string` |
| **exact**          | indicates that the field can also be used to match an exact value by appending `.exact` to name of the field | `true` or not present |
| **multi**          | indicates that the field can contain a single value or an array of values | `true` or not present |
| **sortable**       | indicates the field can be used to sort the the results | `true` or non present |
| **not_searchable** | indicates that the field, can not be used in the filters or query but its value can fetch | `true` or non present |

> **Dates are expressed in milliseconds since Epoch.**

### 2. list

This method can be used to get a list of entity items.

It accepts the following paramaters:

| Parameter | description | values |
| --------- | ----------- | ------ |
| limit | limits the number of items to return. The default is `10` and the maximum `1000`. | `1` to `1000` |
| offset | indicates the offset from which to return the items. It can be used to create a pager. The default is `0`. | >= `0` |
| fields | indicates wich fields to `include` or `exclude` for each item. It can be used to get a partial field. The default field depends on the entity type. See below for more details. | array of field names to `include` or `exclude` |
| query | this is the main parameter to search for particular items. It's a classic search query which accepts an extended syntax. The default is no query. See below for more details. | object |
| filter | this allows to filter the results, it can be a simple filter or a logical combination of filters. The default filter depends on the entity type is overriden when the filter parameter is present. | object |

Example:

```json
curl -XGET 'http://api.rwdev.org/v0/report/list' -d '{
  "offset" : 0,
  "limit" : 3,
  "fields" : {
    "include" : ["title", "primary_country"],
    "exlcude" : ["primary_country.shortname"]
  },
  "query" : {
    "value" : "syria humanitarian",
    "fields" : ["primary_country^5", "title^2", "body"],
    "operator" : "and"
  },
  "filter" : {
    "conditions" : [
      {
        "field" : "status",
        "value" : 1
      },
      {
        "field": "headline"
      }
    ],
    "operator" : "and"
  }
}'
```
