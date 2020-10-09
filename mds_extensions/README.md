

## Table of Contents

- [General Information](#general-information)
- [Aggregate Status Changes](#aggregate-status-changes)
    - [Status Changes - Query Parameters](#status-changes---query-parameters)
    - [Status Changes - Event Types](#status-changes---event-types)
- [Aggregate Trips](#aggregate-trips)
    - [Trips - Query Parameters](#trips---query-parameters)
- [Aggregate Status Counts](#aggregate-status-counts)
    - [Status Counts - Query Parameters](#status-counts---query-parameters)
    - [Status Counts - Event Types](#status-counts---event-types)

## General Information

**Status:** Alpha

APIs described in this document are available for use, but are subject to change without warning. 
Please send questions and comments regarding these APIs to [Lime MDS Team](mailto:mds-team@li.me).

The MDS Extensions APIs will follow the same standards, nomenclature, and data types as stock MDS. 
This includes leveraging the [MDS provider response format](https://github.com/CityOfLosAngeles/mobility-data-specification/tree/dev/provider#response-format), [JSON API pagination](https://jsonapi.org/format/#fetching-pagination), UUIDs for devices 
identification, [GeoJSON for geographic data](https://tools.ietf.org/html/rfc7946), [MDS event type names](https://github.com/openmobilityfoundation/mobility-data-specification/tree/0.3.x/provider#event-types), and [MDS timestamp formatting](https://github.com/CityOfLosAngeles/mobility-data-specification/tree/dev/provider#timestamps).
All endpoints described within this document aggregate data by geospatial area. Data aggregations are subdivided into 
hexagonal areas, which will be generated using the H3 library published by Uber. 

[Top](#Table-of-Contents)

## Aggregate Status Changes

- The aggregate status changes endpoint will return a list of MDS status change counts by event type and event type 
reason within a hexagonal area based on the last known lat/long of the vehicle.
- Each entry in the list will contain the following
  - A summary list, which will contain the event_type, event_type_reason and volume, which is the number of vehicles 
  in that state.
  - A location of GPS coordinates that overlaps with the location's bounding area
- The aggregations will be in one hour segments.
- To ensure k-anonymity, the endpoint will not surface locations where the volume is less than 5.

**Endpoint:** `/aggregate/status_changes`

**HTTP Method:** `GET`

**Data payload format:** `{ "status_changes": [] }`, an array of objects with the following structure:

| Field  | Type | Comments  |
| -----  | ---- | --------  |
| `summary`  | Array | Array of [Event counts](#event-counts).  |
|  `location` |  GeoJSON Point | A [GeoJSON Polygon object](http://wiki.geojson.org/GeoJSON_draft_version_6#Polygon) defining the bounding area. (Currently, all are hexagonal, but may accommodate different shapes based on future implementation changes) The current [resolution](https://uber.github.io/h3/#/documentation/core-library/resolution-table) for hex is 8. |

### Event counts

List of counts by `event_type` and `event_type_reason`

| Field  | Type | Comments  |
| -----  | ---- | --------  |
| `event_type` | String | The event type of the last status change event reported in the MDS feed for the vehicle. See [vehicle status](https://github.com/openmobilityfoundation/mobility-data-specification/tree/0.3.x/provider#event-types) table. |
| `event_type_reason` | String | The reason for the status change of the last status change event reported in the MDS feed for the vehicle. See [vehicle states](https://github.com/openmobilityfoundation/mobility-data-specification/tree/0.3.x/provider#event-types) table.
| `volume` | Integer | The count of vehicles by event_type and event_type_reason during the period of time. |

**Example usage:**
```
# Request

curl \
-H "Accept: application/vnd.mds.provider+json;version=0.3" \
-H "Authorization: Bearer $LIME_TOKEN" \
-X GET \
"https://data.lime.bike/api/partners/v1/mds/{city}/aggregate/status_changes?min_end_time=1598670000000&max_end_time=1598673600000"

# Response

{
    “status_changes”: [
        {
            "summary": [
                {
                    "event_type": "available",
                    "event_type_reason": "user_drop_off",
                    "volume": 30
                },
                {
                    "event_type": "reserved",
                    "event_type_reason": "user_pick_up",
                    "volume": 21
                }
            ],
        "location": {
            "type": "Polygon",
            "coordinates": [ … ]
        }
      },
    ]
}

```

[Top](#Table-of-Contents)

### Status Changes - Query Parameters

The `/aggregate/status_changes` API will allow querying aggregate status changes with the following query parameters:

| Field  | Type | Comments  |
| -----  | ---- | --------  |
| `min_end_time` | [Timestamp](https://en.wikipedia.org/wiki/Unix_time) | filter for `status changes` after the given time.  |
| `max_end_time` | [Timestamp](https://en.wikipedia.org/wiki/Unix_time) | filter for `status changes` before the given time. |

If the timestamps are not hour-bounded, the endpoint will round down to the most recent hour. If not provided, the
endpoint will return the most recent hour for which it has data.

[Top](#Table-of-Contents)

### Status Changes - Event Types

The below table describes the list of event types and reasons as specified in [MDS specification](https://github.com/openmobilityfoundation/mobility-data-specification/blob/0.3.x/provider/README.md#event-types) for which the status_changes endpoint will provide aggregated counts within a hexagonal area based on the last known lat/long of the vehicle.

| `event_type` | Description | `event_type_reason` | Description |
| ---------- | ---------------------- | ------- | ------------------ |
| `available` | A device becomes available for customer use | `user_drop_off` | User ends reservation |
| | | `rebalance_drop_off` | Device moved for rebalancing |
| | | `maintenance_drop_off` | Device introduced into service after being removed for maintenance |
| `reserved` | A customer reserves a device (even if trip has not started yet) | `user_pick_up` | Customer reserves device |
| `unavailable` | A device is on the street but becomes unavailable for customer use | `maintenance` | A device is no longer available due to equipment issues |
| | | `low_battery` | A device is no longer available due to insufficient battery |
| `removed` | A device is removed from the street and unavailable for customer use | `rebalance_pick_up` | Device removed from street and will be placed at another location to rebalance service |
| | | `maintenance_pick_up` | Device removed from street so it can be worked on |

[Top](#Table-of-Contents)

## Aggregate Trips

- The aggregated trips endpoint will return a list of visited locations, where a visited location consists 
of a lat/long coordinate identifying the center of a hexagonally shaped bounded area through which trips have passed.
- Each entry in the list will also contain a volume, which is the number of trips taken by unique users that have a 
GPS coordinate that overlaps with the location’s bounding area.
- To ensure k-anonymity, the trips endpoint will not surface locations where the volume is less than 5.
- For performance reasons, and to maintain anonymity, it is an error to request data from the trips endpoint 
when `max_end_time` and `min_end_time` are less than an hour apart.

**Endpoint:** `/aggregate/trips`

**HTTP Method:** `GET`

**Data payload format:** `{ "trips": [] }`, an array of objects with the following structure:

| Field  | Type | Comments  |
| -----  | ---- | --------  |
| `volume`  | Integer | The count of trips for unique users that have passed through the location’s bounding area during the period of time.  |
|  `location` |  GeoJSON Point | A [GeoJSON Polygon object](http://wiki.geojson.org/GeoJSON_draft_version_6#Polygon) defining the bounding area. (Currently, all are hexagonal, but may accommodate different shapes based on future implementation changes.) The current [resolution](https://uber.github.io/h3/#/documentation/core-library/resolution-table) for hex is 8. |

**Example usage:**
```
# Request

curl \
-H "Accept: application/vnd.mds.provider+json;version=0.3" \
-H "Authorization: Bearer $LIME_TOKEN" \
-X GET \
"https://data.lime.bike/api/partners/v1/mds/{city}/aggregate/trips?min_end_time=1598670000000&max_end_time=1598673600000"

# Response

{	
	“trips”: [ 
        {
    		"volume": 25,
    		"location": {
                "type": "Polygon",
                "coordinates": [...]
    		}
  	    }
    ]
}

```

[Top](#Table-of-Contents)

### Trips - Query Parameters

The `/aggregate/trips` API will allow querying aggregate trips with the following query parameters:

| Field  | Type | Comments  |
| -----  | ---- | --------  |
| `min_end_time` | [Timestamp](https://en.wikipedia.org/wiki/Unix_time) | filter for `trips` after the given time.  |
| `max_end_time` | [Timestamp](https://en.wikipedia.org/wiki/Unix_time) | filter for `trips` before the given time. |

If the timestamps are not hour-bounded, the endpoint will round down to the most recent hour. If not provided, the
endpoint will return the most recent hour for which it has data.

[Top](#Table-of-Contents)

## Aggregate Status Counts

- The aggregate status counts endpoint will return a list of MDS status change counts by [event type](https://github.com/openmobilityfoundation/mobility-data-specification/tree/0.3.x/provider#event-types)
  based on last reported status changes for vehicles in the specified sliding window.
- The `data` property will provide information about `vehicle_type`.
  - A `vehicle_type` list , will contain the hour_utc and event_type(s) count, which is the number of vehicles 
    in that state.
- The aggregations will be in one hour segments.

**Endpoint:** `/aggregate/status_counts`

**HTTP Method:** `GET`

**Data payload format:** `{ "data": { "scooter": [] } }`, an array of objects with the following structure:

| Field  | Type | Comments  |
| -----  | ---- | --------  |
| `data`  | Objects | Holds the status counts for each `vehicle type`.  |
|  `lookback_days` |  Integer | Number of days to look back for vehicle status. |

| Field  | Type | Comments  |
| -----  | ---- | --------  |
| `hour_utc`  | datetime [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) | Specifies the datetime when status count aggregation was performed.  |
| `lookback_days` |  Integer | Number of days to look back for vehicle status, `default:` **2d** |

**Example usage:**
```
# Request

curl \
-H "Accept: application/vnd.mds.provider+json;version=0.3" \
-H "Authorization: Bearer $LIME_TOKEN" \
-X GET \
"https://data.lime.bike/api/partners/v1/mds/{city}/aggregate/status_counts?min_end_time=1602176400000&max_end_time=1602183600000&lookback_days=2"

# Response

{
    "data": {
        "scooter": [
            {
                "hour_utc": "2020-10-08T17:00:00.000Z",
                "available": 156,
                "reserved": 31,
                "unavailable": 37,
                "removed": 218
            },
            {
                "hour_utc": "2020-10-08T18:00:00.000Z",
                "available": 150,
                "removed": 218,
                "unavailable": 36,
                "reserved": 34
            }
        ]
    },
    "lookback_days": 2
}

```

[Top](#Table-of-Contents)

### Status Counts - Query Parameters

The `/aggregate/status_counts` API will allow querying aggregate status_counts with the following query parameters:

| Field  | Type | Comments  |
| -----  | ---- | --------  |
| `min_end_time` | [Timestamp](https://en.wikipedia.org/wiki/Unix_time) | filter for `status_counts` after the given time.  |
| `max_end_time` | [Timestamp](https://en.wikipedia.org/wiki/Unix_time) | filter for `status_counts` before the given time. |
| `lookback_days`| Integer | Number of days to look back for vehicle status.  **Note:** Vehicles that have not had an event in this number of days will not appear in the counts. |

If the timestamps are not hour-bounded, the endpoint will round down to the most recent hour. If not provided, the
endpoint will return the most recent hour for which it has data.

[Top](#Table-of-Contents)

### Status Counts - Event Types

The below table describes the list of event types as specified in [MDS specification](https://github.com/openmobilityfoundation/mobility-data-specification/blob/0.3.x/provider/README.md#event-types) for which the status_counts endpoint will provide aggregated counts.

| `event_type` | Description | 
| ---------- | ---------------------- |
| `available` | A device becomes available for customer use | 
| `reserved` | A customer reserves a device (even if trip has not started yet) |
| `unavailable` | A device is on the street but becomes unavailable for customer use |
| `removed` | A device is removed from the street and unavailable for customer use | 

[Top](#Table-of-Contents)