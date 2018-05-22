# MobilizeAmerica API Pilot

**Note: This API is in alpha development, and while we will strive to minimize breaking changes and keep this document up-to-date, we expect there will be some iteration before we land on a public stable version.**

Please see the [Changelog](#changelog) below for changes, and file any issues [here](https://github.com/mobilizeamerica/api/issues)

To stay updated on new releases or iterations, join the email list [here](https://groups.google.com/forum/#!forum/mobilizeamerica-api-users)

# Overview
## API Entry Point

All requests occur through `events.mobilizeamerica.io/api/v1`.

## Format

APIs will return responses in JSON, with the following shared format.

| Field      | Type         | Description                                                                                                                              |
| ---------- | ------------ | ---------------------------------------------------------------------------------------------------------------------------------------- |
| `data`     | array|object | Objects returned by the API (on this page).                                                                                              |
| `error`    | object       | Any errors with input or errors on the server. If this key is present, paging related keys will not be present, and data will be `null`. |
| `count`    | int          | If a paged response, the number of objects across all pages.                                                                             |
| `next`     | url          | The next page if it exists, otherwise `null`.                                                                                            |
| `previous` | url          | The previous page if it exists, otherwise `null`.                                                                                        |

## Authentication

For API routes which require authentication (not all do) you must access the API with a key which we will provide you. This key should be passed as the token in a bearer auth header, i.e.:


    Authorization: Bearer API_KEY

All requests must be made over HTTPS. Requests made with malformed, missing or otherwise incorrect API keys will return HTTP status 403.

## Errors

Error responses will have an object value for the `error` field and `data` will be `null`. We will also return a relevant HTTP status code with each error response.

Here’s an example error response for `/api/v1/events?organization_id=hello`:


    {
        "data": null,
        "error": {
            "organization_id": {
                "0": [
                    "A valid integer is required."
                ]
            }
        }
    }
    
## Paging

All endpoints support paging, using the query params `per_page` and `page`. `per_page` defaults to 25. In the response, `count` describes the number of total objects, so `ceiling(count / per_page)` gives the total number of pages. Also, for convenience, `next` and `previous` will be links to the next and previous pages respectively.

## Comparison Filters

Some endpoints will allow for filtering by comparing with a field within an object. In those cases, the format will be `field_name=cmp_####`. For example, to filter out events before Jan 1 2018 GMT, you can include `timeslot_start=gte_1514764800`. Multiple may also be used, e.g. `timeslot_start=gte_1514764800&timeslot_start=lt_1515110400`. The comparison operators are ≥ `gte`, > `gt`, ≤ `lte`, < `lt`. More may be added in the future.

## OSDI compliance

OSDI is an exciting and important attempt to bring interoperability to the progressive data ecosystem, a cause we heartily support. However, there are a number of data model and format constraints of OSDI that made it a less-than-perfect fit for us and our consumers. While we have opted not to follow strict OSDI formats for our API responses at this stage, we have made our best effort to align field names and structure where possible. For example, many of the Event fields map directly onto OSDI field names. We hope this will make the job of anyone attempting to build an OSDI layer on top of our API easier.

# Organizations
## Organization object

| Field                 | Type   | Description                                                                                                                                                                                   |
| --------------------- | ------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `id`                  | int    |                                                                                                                                                                                               |
| `name`                | string | The public-facing name of the organization.                                                                                                                                                   |
| `slug`                | string | The URL-safe string for this organization.                                                                                                                                                    |
| `is_coordinated`      | bool   | Whether this is a coordinated-side organization for campaign finance purposes. Opposite of `is_independent`                                                                                   |
| `is_independent`      | bool   | Whether this is an independent-side organization for campaign finance purposes. Opposite of `is_coordinated`                                                                                  |
| `race_type`           | enum   | One of `GOVERNOR`, `CONGRESSIONAL`, `SENATE`, `STATE_SENATE`, `STATE_LEG`, `SEC_STATE`, `ATTY_GENERAL`, `OTHER_LOCAL`, `OTHER_STATEWIDE`. New options may be added. `null` if not a campaign. |
| `is_primary_campaign` | bool   | If the campaign is a primary. `false` if not a campaign.                                                                                                                                      |
| `state`               | string | The two-letter code of the state the organization is in. May be empty.                                                                                                                        |
| `district`            | string | The district the organization is in, e.g., `14`. May refer to different districts depending on the value of `race_type`.                                                                      |
| `candidate_name`      | string | The name of the candidate this organization represents. May be empty.                                                                                                                         |
| `event_feed_url`      | string | The public event feed for this organization.                                                                                                                                                  |
| `created_date`        | int    | Unix timestamp                                                                                                                                                                                |
| `modified_date`       | int    | Unix timestamp                                                                                                                                                                                |

## List all organizations

Status: LIVE

Return all active organizations on the platform. This endpoint is publicly accessible.

Requires authentication: No

### Request
`GET /api/v1/organizations`

### Request Params

- `updated_since`: Unix timestamp of organizations to filter by `modified_date`

### Response
`data` is an array of Organization objects.

    {
      ...,
      "data": [
        {
          "id": 1,
          "name": "Example",
          "slug": "example",
          "is_coordinated": true,
          "is_independent": false,
          "is_primary_campaign": false,
          "state": "",
          "district": "",
          "candidate_name": "",
          "race_type": null,
          "event_feed_url": "https://events.mobilizeamerica.io/example/",
          "created_date": 1524232122,
          "modified_date": 1524232122
        },
        ...
      ]
    }


## List all the organizations promoted by an organization

Status: LIVE

Fetches a list of all the organizations that an organization has promoted. This endpoint is accessible only to members of the promoting organization.

Requires authentication: Yes

### Request
`GET /api/v1/organizations/:organization_id/promoted_organizations`

### Request Params

- None

### Response
`data` is an array of Organization objects.

    {
      ...,
      "data": [
        {
          "id": 1,
          "name": "Example Organization",
          "slug": "example",
          "is_coordinated": true,
          "is_independent": false,
          "is_primary_campaign": false,
          "state": "",
          "district": "",
          "candidate_name": "",
          "race_type": null,
          "event_feed_url": "https://events.mobilizeamerica.io/example/",
          "created_date": 1524232122,
          "modified_date": 1524232122
        },
        ...
      ]
    }

# Events
## Event object
| Field                | Type         | Description                                                                                                                                             |
| -------------------- | ------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `id`                 | int          |                                                                                                                                                         |
| `title`              | string       | The public name of the event                                                                                                                            |
| `summary`            | string       | The public subheading of the event                                                                                                                      |
| `description`        | string       | Long-form HTML description of the event                                                                                                                 |
| `featured_image_url` | string       | Path to the image for the event                                                                                                                         |
| `sponsor`            | Organization | The owning Organization, with the fields defined above                                                                                                  |
| `timeslots`          | Timeslot[]   | Array of past and future timeslots                                                                                                                      |
| `location`           | Location     | The event location, or `null` if the event is virtual                                                                                                   |
| `timezone`           | string       | A timezone database string for the event, e.g., `America/New_York`.                                                                                     |
| `event_type`         | enum         | The type of the event, one of: `CANVASS`, `PHONE_BANK`, `TEXT_BANK`, `MEETING`, `COMMUNITY`, `FUNDRAISER`, `MEET_GREET`, `OTHER`. This list may expand. |
| `browser_url`        | string       | Canonical URL of the event                                                                                                                              |
| `created_date`       | int          | Unix timestamp                                                                                                                                          |
| `modified_date`      | int          | Unix timestamp                                                                                                                                          |


### Timeslot

| Field        | Type | Description    |
| ------------ | ---- | -------------- |
| `id`         | int  |                |
| `start_date` | int  | Unix timestamp |
| `end_date`   | int  | Unix timestamp |


### Location

| Field                    | Type     | Description                                                                                                |
| ------------------------ | -------- | ---------------------------------------------------------------------------------------------------------- |
| `venue`                  | string   | The name of the location, e.g., “Campaign HQ” or “Starbucks”                                               |
| `address_lines`          | string[] | The lines of the address. Should always have exactly two values in our system, which may be empty strings. |
| `locality`               | string   | The city                                                                                                   |
| `region`                 | string   | The two-character state code                                                                               |
| `postal_code`            | string   | The zipcode                                                                                                |
| `location`               | object   | The geocoded location, or `null` if geocoding failed.                                                      |
| `location.latitude`      | float    |                                                                                                            |
| `location.longitude`     | float    |                                                                                                            |

### Deleted Event
| Field          | Type | Description    |
| -------------- | ---- | -------------- |
| `id`           | int  |                |
| `deleted_date` | int  | Unix timestamp |

## List all public events

Status: LIVE

Fetch all public events on the platform.

Requires authentication: No

### Request
`GET /api/v1/events`

### Request params

- `organization_id`: One or more Organization IDs to filter to. If multiple, should be supplied as multiple query params, e.g., `organization_id=1&organization_id=2`, etc.
- `updated_since`: Unix timestamp to filter by Events’ `modified_date`
- `timeslot_start`: Comparison to filter by Events' Timeslots' start date. Will only return Timeslots on those Events that meet the filter conditions

### Response
`data` is an array of Event objects.

    {
      ...,
      "data": [
        {
          "id": 1,
          "description": "example",
          "timezone": "America/New_York",
          "title": "Example",
          "summary": "",
          "featured_image_url": "",
          "sponsor": {
            Organization object
          },
          "timeslots": [
              {
                  "id": 1,
                  "start_date": 2,
                  "end_date": 3
              },
              {
                  "id": 2,
                  "start_date": 3,
                  "end_date": 4
              }
          ],
          "location": {
              "venue": "",
              "address_lines": [
                  "",
                  ""
              ],
              "locality": "",
              "region": "",
              "postal_code": "10003",
              "location": {
                  "latitude": 40.7322535,
                  "longitude": -73.9874105
              }
          },
          "event_type": "CANVASS",
          "created_date": 1,
          "modified_date": 1,
          "browser_url": "https://events.mobilizeamerica.io/event/1/"
        },
        ...
      ]
    }   

## List organization events

Status: LIVE

Fetch all public events for an organization. This includes both events owned by the organization (as indicated by the `organization` field on the event object) and events of other organizations promoted by this specified organization.

Requires authentication: No

### Request
`GET /api/v1/organizations/:organization_id/events`

### Request params

- `updated_since`: Unix timestamp to filter by Events’ `modified_date`
- `timeslot_start`: Comparison to filter by Events' Timeslots' start date. Will only return Timeslots on those Events that meet the filter conditions

### Response
`data` is an array of Event objects.

    {
      ...,
      "data": [
        {
          "id": 1,
          "description": "example",
          "timezone": "America/New_York",
          "title": "Example",
          "summary": "",
          "featured_image_url": "",
          "sponsor": {
            Organization object
          },
          "timeslots": [
              {
                  "id": 1,
                  "start_date": 2,
                  "end_date": 3
              },
              {
                  "id": 2,
                  "start_date": 3,
                  "end_date": 4
              }
          ],
          "location": {
              "venue": "",
              "address_lines": [
                  "",
                  ""
              ],
              "locality": "",
              "region": "",
              "postal_code": "10003",
              "location": {
                  "latitude": 40.7322535,
                  "longitude": -73.9874105
              }
          },
          "event_type": "CANVASS",
          "created_date": 1,
          "modified_date": 1,
          "browser_url": "https://events.mobilizeamerica.io/event/1/"
        },
        ...
      ]
    }

## List deleted public events

Status: LIVE

Fetch deleted public events on the platform.

Requires authentication: No

### Request
`GET /api/v1/events/deleted`

### Request params

- `organization_id`: One or more Organization IDs to filter to. If multiple, should be supplied as multiple query params, e.g., `organization_id=1&organization_id=2`, etc.
- `updated_since`: Unix timestamp to filter by Events’ `modified_date`

### Response
`data` is an array of Deleted Event objects.

## List deleted organization events

Status: LIVE

Fetch all deleted public events for an organization. This includes both events owned by the organization (as indicated by the `organization` field on the event object) and events of other organizations promoted by this specified organization.

Requires authentication: No

### Request
`GET /api/v1/organizations/:organization_id/events/deleted`

### Request params

- `updated_since`: Unix timestamp to filter by Events’ `modified_date`

### Response
`data` is an array of Deleted Event objects.

# People
## Person object
| Field              | Type      | Description                           |
| ------------------ | --------- | ------------------------------------- |
| `id`               | int       |                                       |
| `created_date`     | int       | Unix timestamp                        |
| `modified_date`    | int       | Unix timestamp                        |
| `given_name`       | string    | First name                            |
| `family_name`      | string    | Last name                             |
| `email_addresses`  | Email[]   | Array of length 1 with an email       |
| `phone_numbers`    | Phone[]   | Array of length 1 with a phone number |
| `postal_addresses` | Address[] | Array of length 1 with a zip code     |

### Email

| Field     | Type   | Description   |
| --------- | ------ | ------------- |
| `primary` | bool   | Always true   |
| `address` | string | Email address |

### Phone

| Field     | Type   | Description  |
| --------- | ------ | ------------ |
| `primary` | bool   | Always true  |
| `number`  | string | Phone number |

### Address

| Field         | Type   | Description |
| ------------- | ------ | ----------- |
| `primary`     | bool   | Always true |
| `postal_code` | string | Zip code    |

## List organization people

Status: LIVE

Fetch all people (volunteers) who are affiliated with the organization

Requires authentication: Yes

### Request
`GET /api/v1/organizations/:organization_id/people`

### Request params

- `updated_since`: Unix timestamp to filter by Persons’ `modified_date`

### Response
`data` is an array of Person objects.

# Attendances
## Attendance object

| Field           | Type         | Description                                                                 | Coordinated/Independent notes                                                                                               |
| --------------- | ------------ | --------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| `id`            | int          |                                                                             |                                                                                                                             |
| `created_date`  | int          | Unix timestamp                                                              |                                                                                                                             |
| `modified_date` | int          | Unix timestamp                                                              |                                                                                                                             |
| `person`        | Person       | Person who attended the event                                               |                                                                                                                             |
| `event`         | Event        | Associated event                                                            | If the requesting organization is independent and the event’s organization is coordinated, all but `event_type` is omitted.  |
| `timeslot`      | Timeslot     | Selected timeslot on event                                                  | If the requesting organization is independent and the event’s organization is coordinated, `id` is omitted.              |
| `sponsor`       | Organization | The promoting organization if it exists, otherwise the event’s organization | If the requesting organization is coordinated and the promoting organization is independent, this is omitted.              |
| `status`        | enum         | `REGISTERED`, `CANCELLED`, or `CONFIRMED`                                   |                                                                                                                             |
| `attended`      | bool         | Whether the person actually attended or not                                 |                                                                                                                             |
| `referrer`      | Referrer     | UTM tracking information                                                    |                                                                                                                             |

### Referrer

| Field          | Type   | Description |
| -------------- | ------ | ----------- |
| `utm_source`   | string |             |
| `utm_medium`   | string |             |
| `utm_campaign` | string |             |
| `utm_term`     | string |             |
| `utm_content`  | string |             |
| `url`          | string |             |

## List organization attendances

Status: LIVE

Fetch all attendances which were either promoted by the organization or were for events owned by the organization

Requires authentication: Yes

### Request
`GET /api/v1/organizations/:organization_id/attendances`

### Request params

- `updated_since`: Unix timestamp to filter by Attendances’ `modified_date`

### Response
`data` is an array of Attendance objects.

## List organization’s affiliated person’s attendances

Status: LIVE

Fetches all attendances that are either for that person with that organization, or are for public events and were created after the affiliation between the person and the organization began.

Requires authentication: Yes

### Request
`GET /api/v1/organizations/:organization_id/people/:person_id/attendances`

### Request params

- `updated_since`: Unix timestamp to filter by Attendances’ `modified_date`

### Response
`data` is an array of Attendance objects.


# Changelog

**2018-05-23**
- Add `attended` to Attendance object

**2018-05-08**

- Add endpoint for deleted events
- Add endpoint for deleted events for an organization and its promoted organizations
- Add filtering by comparisons for timeslot start dates on events

**2018-05-03**

- Add endpoint to fetch promoted organizations 
- Add endpoint to fetch affiliated people
- Add endpoint to fetch attendences for an organization
- Add endpoint to fetch attendences for a person
- Added `url` to Referrer object
