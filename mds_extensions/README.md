

## Table of Contents

- [General Information](#general-information)
- [Aggregate Status Changes](#aggregate-status-changes)
    - [Status Changes - Query Parameters](#status-changes---query-parameters)
- [Aggregate Trips](#aggregate-trips)
    - [Trips - Query Parameters](#trips---query-parameters)

## General Information

**Status:** Alpha
APIs described in this document are available for use, but are subject to change without warning. 
Please send questions and comments regarding these APIs to [Lime MDS Team](mailto:mds-team@li.me).

The MDS Extensions APIs will follow the same standards, nomenclature, and data types as stock MDS. 
This includes leveraging the [MDS provider response format](https://github.com/CityOfLosAngeles/mobility-data-specification/tree/dev/provider#response-format), [JSON API pagination](https://jsonapi.org/format/#fetching-pagination), UUIDs for devices 
identification, [GeoJSON for geographic data](https://tools.ietf.org/html/rfc7946), [MDS event type names](https://github.com/CityOfLosAngeles/mobility-data-specification/tree/dev/provider#event-types), and [MDS timestamp formatting](https://github.com/CityOfLosAngeles/mobility-data-specification/tree/dev/provider#timestamps).
All endpoints described within this document aggregate data by geospatial area. Data aggregations are subdivided into 
hexagonal areas, which will be generated using the H3 library published by Uber. 

[Top][toc]

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
| `summary`  | Array | Array of [Event counts](#event-counts))  |
|  `location` |  GeoJSON Point | A [GeoJSON Polygon object](http://wiki.geojson.org/GeoJSON_draft_version_6#Polygon) defining the bounding area. (Currently, all are hexagonal, but may accommodate different shapes based on future implementation changes.) The current [resolution](https://uber.github.io/h3/#/documentation/core-library/resolution-table) for hex is 8) |
|   |   |   |

### Event counts

List of counts by `event_type` and `event_type_reason`

| Field  | Type | Comments  |
| -----  | ---- | --------  |
| `event_type` | String | The event type of the last status change event reported in the MDS feed for the vehicle. See [vehicle status](https://github.com/openmobilityfoundation/mobility-data-specification/blob/dev/general-information.md#vehicle-state-events) table. |
| `event_type_reason` | String | The reason for the status change of the last status change event reported in the MDS feed for the vehicle. See [vehicle states](https://github.com/openmobilityfoundation/mobility-data-specification/blob/dev/general-information.md#vehicle-state-events) table.
| `volume` | Integer | The count of vehicles by event_type and event_type_reason during the period of time |

[Top][toc]

### Status Changes - Query Parameters

The `/aggregate/status_changes` API will allow querying aggregate status changes with the following query parameters:

[Top][toc]

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

[Top][toc]

### Trips - Query Parameters

[Top][toc]