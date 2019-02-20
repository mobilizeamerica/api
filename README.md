# MobilizeAmerica API Pilot

**Note: This API is in alpha development, and while we will strive to minimize breaking changes and keep this document up-to-date, we expect there will be some iteration before we land on a public stable version.**

Please see the [Changelog](#changelog) below for changes, and file any issues [here](https://github.com/mobilizeamerica/api/issues)

To stay updated on new releases or iterations, join the email list [here](https://groups.google.com/forum/#!forum/mobilizeamerica-api-users)

# Table of Contents
- [MobilizeAmerica API Pilot](#mobilizeamerica-api-pilot)
- [Table of Contents](#table-of-contents)
- [Overview](#overview)
    - [API Entry Point](#api-entry-point)
    - [Format](#format)
    - [Authentication](#authentication)
    - [Errors](#errors)
    - [Paging](#paging)
    - [Comparison Filters](#comparison-filters)
    - [OSDI compliance](#osdi-compliance)
- [Organizations](#organizations)
    - [Organization object](#organization-object)
    - [List all organizations](#list-all-organizations)
        - [Request](#request)
        - [Request Params](#request-params)
        - [Response](#response)
    - [List all the organizations promoted by an organization](#list-all-the-organizations-promoted-by-an-organization)
        - [Request](#request-1)
        - [Request Params](#request-params-1)
        - [Response](#response-1)
- [Events](#events)
    - [Event object](#event-object)
        - [Timeslot](#timeslot)
        - [Location](#location)
        - [Contact](#contact)
        - [Deleted Event](#deleted-event)
    - [List all public events](#list-all-public-events)
        - [Request](#request-2)
        - [Request params](#request-params-2)
        - [Response](#response-2)
    - [List organization events](#list-organization-events)
        - [Request](#request-3)
        - [Request params](#request-params-3)
        - [Response](#response-3)
    - [List deleted public events](#list-deleted-public-events)
        - [Request](#request-4)
        - [Request params](#request-params-4)
        - [Response](#response-4)
    - [List deleted organization events](#list-deleted-organization-events)
        - [Request](#request-5)
        - [Request params](#request-params-5)
        - [Response](#response-5)
    - [Create event](#create-event)
        - [Request](#request-6)
        - [Request body](#request-body)
        - [Request body example](#request-body-example)
        - [Response](#response-6)
    - [Update event](#update-event)
        - [Request](#request-7)
        - [Request params](#request-params-6)
        - [Request body](#request-body-1)
        - [Request body example](#request-body-example-1)
        - [Response](#response-7)
    - [Delete event](#delete-event)
        - [Request](#request-8)
        - [Request params](#request-params-7)
        - [Response](#response-8)
- [People](#people)
    - [Person object](#person-object)
        - [Email](#email)
        - [Phone](#phone)
        - [Address](#address)
    - [List organization people](#list-organization-people)
        - [Request](#request-9)
        - [Request params](#request-params-8)
        - [Response](#response-9)
- [Attendances](#attendances)
    - [Attendance object](#attendance-object)
        - [Referrer](#referrer)
    - [List organization attendances](#list-organization-attendances)
        - [Request](#request-10)
        - [Request params](#request-params-9)
        - [Response](#response-10)
    - [Create organization event attendance](#create-organization-event-attendance)
        - [Request](#request-11)
        - [Request body](#request-body-2)
        - [Person attendance object](#person-attendance-object)
        - [Referrer object](#referrer-object)
        - [Example request body](#example-request-body)
        - [Response](#response-11)
        - [Response Body Example](#response-body-example)
    - [List organization’s affiliated person’s attendances](#list-organizations-affiliated-persons-attendances)
        - [Request](#request-12)
        - [Request params](#request-params-10)
        - [Response](#response-12)
- [Affiliation](#affiliation)
    - [Create organization affiliations](#create-organization-affiliations)
        - [Request](#request-13)
        - [Request body](#request-body-3)
        - [Request body example](#request-body-example-2)
        - [Response](#response-13)
- [Changelog](#changelog)

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

For timestamp comparisons, the string "now" can be passed in place of a Unix timestamp to have the server compute the current time, e.g., `timeslot_start=gte_now`.

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
| `high_priority`      | bool         | Whether the event is marked high priority for the provided organization                                                                                 |
| `sponsor`            | Organization | The owning Organization, with the fields defined above                                                                                                  |
| `timeslots`          | Timeslot[]   | Array of past and future timeslots                                                                                                                      |
| `location`           | Location     | The event location, or `null` if the event is virtual                                                                                                   |
| `timezone`           | string       | A timezone database string for the event, e.g., `America/New_York`.                                                                                     |
| `event_type`         | enum         | The type of the event, one of: `CANVASS`, `PHONE_BANK`, `TEXT_BANK`, `MEETING`, `COMMUNITY`, `FUNDRAISER`, `MEET_GREET`, `HOUSE_PARTY`, `VOTER_REG`, `TRAINING`, `FRIEND_TO_FRIEND_OUTREACH`, `OTHER`. This list may expand. |
| `browser_url`        | string       | Canonical URL of the event                                                                                                                              |
| `created_date`       | int          | Unix timestamp                                                                                                                                          |
| `modified_date`      | int          | Unix timestamp                                                                                                                                          |
| `visibility`         | enum         | The visibility of the event, one of: `PUBLIC`, `PRIVATE`.                                                                                               |
| `created_by_distributed_organizing`      | bool          | Whether the event was created using our distributed organizing tool or not                                                                                                                                          |

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
| `congressional_district` | string   | The Congressional district, or `null` if geocoding failed or no street address was provided                |
| `state_leg_district`     | string   | The State Lower House district, or `null` if geocoding failed or no street address was provided            |
| `state_senate_district`  | string   | The State Upper House/Senate district, or `null` if geocoding failed or no street address was provided     |


### Contact
The Contact object will return `null` for unauthenticated requests.

| Field            | Type    | Description                                     |
| ---------------- | ------- | ----------------------------------------------- |
| `name`           | string  | The name of the contact for the event.          |
| `email_adddress` | string  | The email address of the contact for the event. |
| `phone_number`   | string  | The phone number of the contact for the event.  |


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
- `timeslot_end`: Comparison to filter by Events' Timeslots' end date. Will only return Timeslots on those Events that meet the filter conditions
- `zipcode`: Zipcode to filter by Events' Locations' postal code. If present, will return Events sorted by distance from zipcode. When zipcode is provided, virtual events will not be returned.
- `max_dist`: Maximum distance (in miles) to filter by Events' Locations' distance from provided zipcode.

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
          "high_priority": null,
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
                  "204 E 13th St",
                  ""
              ],
              "locality": "",
              "region": "",
              "postal_code": "10003",
              "location": {
                  "latitude": 40.7322535,
                  "longitude": -73.9874105
              },
              "congressional_district_value": "12",
              "state_leg_district_value": "66",
              "state_senate_district_value": "27",
          },
          "event_type": "CANVASS",
          "created_date": 1,
          "modified_date": 1,
          "browser_url": "https://events.mobilizeamerica.io/event/1/"
        },
         "contact": {
          "name": "",
          "email_address": "replyto@thisemail.org",
          "phone_number": "1234567890"
        },
        ...
      ]
    }

## List organization events

Status: LIVE

Fetch all events for an organization. This includes both events owned by the organization (as indicated by the `organization` field on the event object) and events of other organizations promoted by this specified organization. By default, this endpoint will return only public events.

Requires authentication: No

### Request
`GET /api/v1/organizations/:organization_id/events`

### Request params

- `updated_since`: Unix timestamp to filter by Events’ `modified_date`
- `timeslot_start`: Comparison to filter by Events' Timeslots' start date. Will only return Timeslots on those Events that meet the filter conditions
- `timeslot_end`: Comparison to filter by Events' Timeslots' end date. Will only return Timeslots on those Events that meet the filter conditions
- `zipcode`: Zipcode to filter by Events' Locations' postal code. If present, will return Events sorted by distance from zipcode. When zipcode is provided, virtual events will not be returned.
- `max_dist`: Maximum distance (in miles) to filter by Events' Locations' distance from provided zipcode.
- `visibility`: Type of event visibility to filter by; either `PUBLIC` or `PRIVATE`. Private events will only be returned if the calling user is authenticated and has permission to view the given organization's private events in the dashboard. If `visibility=PRIVATE` is specified and the calling user does not have permission, no events will be returned.

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
          "high_priority": true,
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
                  "204 E 13th St",
                  ""
              ],
              "locality": "",
              "region": "",
              "postal_code": "10003",
              "location": {
                  "latitude": 40.7322535,
                  "longitude": -73.9874105
              },
              "congressional_district_value": "12",
              "state_leg_district_value": "66",
              "state_senate_district_value": "27",
          },
          "event_type": "CANVASS",
          "created_date": 1,
          "modified_date": 1,
          "browser_url": "https://events.mobilizeamerica.io/event/1/"
        },
         "contact": {
          "name": "",
          "email_address": "replyto@thisemail.org",
          "phone_number": "1234567890"
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
- `zipcode`: Zipcode to filter by Events' Locations' postal code. If present, will return Events sorted by distance from zipcode. When zipcode is provided, virtual events will not be returned.
- `max_dist`: Maximum distance (in miles) to filter by Events' Locations' distance from provided zipcode.
- `visibility`: Type of event visibility to filter by; either `PUBLIC` or `PRIVATE`. Private events will only be returned if the calling user is authenticated and has permission to view the given organization's private events in the dashboard. If `visibility=PRIVATE` is specified and the calling user does not have permission, no events will be returned.

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
- `zipcode`: Zipcode to filter by Events' Locations' postal code. If present, will return Events sorted by distance from zipcode. When zipcode is provided, virtual events will not be returned.
- `max_dist`: Maximum distance (in miles) to filter by Events' Locations' distance from provided zipcode.

### Response
`data` is an array of Deleted Event objects.

## Create event

Status: RESTRICTED

Please email support@mobilizeamerica.io to request access to this endpoint.

Create a new in-person Event for an given organization.

Requires authentication: Yes

### Request
`POST /api/v1/organizations/:organization_id/events`

### Request body

| Field                | Type         | Description                                                                                                                                             | Required |
| -------------------- | ------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------- | --------
| `title`              | string       | The public name of the event                                                                                                                                                                            | Yes
| `summary`            | string       | The public subheading of the event                                                                                                                                                                      | No
| `description`        | string       | Long-form HTML description of the event                                                                                                                                                                 | Yes
| `timeslots`          | Timeslot[]   | Array of past and future timeslots. Timeslots must be in Unix time.                                                                                                                                     | Yes
| `location`           | Location     | The event location. Note that only in-person events can be created using the public API at this time. `postal_code` is a required field in the `location` object; all other `location` fields are optional.   | Yes
| `timezone`           | string       | A timezone database string for the event, one of: `America/New_York`, `Pacific/Honolulu`, `America/Los_Angeles`, `America/Denver`, `America/Phoenix`, `America/Chicago`.                                | Yes
| `event_type`         | string       | The type of the event, one of: `CANVASS`, `PHONE_BANK`, `TEXT_BANK`, `MEETING`, `COMMUNITY`, `FUNDRAISER`, `MEET_GREET`, `HOUSE_PARTY`, `VOTER_REG`, `TRAINING`, `FRIEND_TO_FRIEND_OUTREACH`, `OTHER`.  | Yes
| `visibility`         | string       | The visibility of the event, one of: `PUBLIC`, `PRIVATE`.                                                                                                                                               | Yes

### Request body example

    {
        "title": "Example",
        "description": "example",
        "timezone": "America/New_York",
        "summary": "",
        "timeslots": [
            {
                "start_date": 1537986600,
                "end_date": 1537986601
            },
            {
                "start_date": 1537986600,
                "end_date": 1537986601
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
            "postal_code": "10003"
        },
        "event_type": "CANVASS",
        "visibility": "PUBLIC",
        "contact": {
            "email_address": "replyto@thisemail.com"
        }
    }

### Response
`data` will contain the newly created Event object.

## Update event

Status: RESTRICTED

Please email support@mobilizeamerica.io to request access to this endpoint.

Update an in-person event for an organization. `event_id` refers to the `id` field in the Event object.

Note that all editable fields must be specified, otherwise they will be removed (e.g. an existing `timeslot`) or set to null (e.g. the `summary` field).

Requires authentication: Yes

### Request
`PUT /api/v1/organizations/:organization_id/events/:event_id`

### Request Params
* `send_update_notifications`: Defaults to `true`. Set this to `false` to skip notifications. Defaults to sending email notifications to attendees when `location` or `timeslots` are updated.
  * `PUT /api/v1/organizations/:organization_id/events/:event_id?send_update_notifications=false`

### Request body
| Field                | Type         | Description                                                                                                                                             | Required |
| -------------------- | ------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------- | --------
| `title`              | string       | The public name of the event                                                                                                                                                                            | Yes
| `summary`            | string       | The public subheading of the event                                                                                                                                                                      | No
| `description`        | string       | Long-form HTML description of the event                                                                                                                                                                 | Yes
| `timeslots`          | Timeslot[]   | Array of past and future timeslots. Timeslots must be a valid Unix timestamp. Any existing timeslots that are not present in the `timeslots` array will be deleted. Existing timeslots may be updated by including the `id` field in the `timeslot` object.                                                                                                                                     | Yes
| `location`           | Location     | The event location. Note that only in-person events can be created using the public API at this time. `postal_code` is a required field in the `location` object.                                       | Yes
| `timezone`           | string       | A timezone database string for the event, one of: `America/New_York`, `Pacific/Honolulu`, `America/Los_Angeles`, `America/Denver`, `America/Phoenix`, `America/Chicago`.                                | Yes
| `event_type`         | string       | The type of the event, one of: `CANVASS`, `PHONE_BANK`, `TEXT_BANK`, `MEETING`, `COMMUNITY`, `FUNDRAISER`, `MEET_GREET`, `HOUSE_PARTY`, `VOTER_REG`, `TRAINING`, `FRIEND_TO_FRIEND_OUTREACH`, `OTHER`.  | Yes
| `visibility`         | string       | The visibility of the event, one of: `PUBLIC`, `PRIVATE`.                                                                                                                                               | Yes

### Request body example

    {
        "title": "Example",
        "description": "example",
        "timezone": "America/New_York",
        "summary": "Short summary for my event",
        "timeslots": [
            {
                "id": 2000,
                "start_date": 1537986600,
                "end_date": 1537986601
            },
            {
                "start_date": 1537986600,
                "end_date": 1537986601,
            }
        ],
        "location": {
            "venue": "Meet at Central Park",
            "address_lines": [
                "",
                ""
            ],
            "locality": "New York City",
            "region": "NY",
            "postal_code": "10003"
        },
        "event_type": "CANVASS",
        "visibility": "PUBLIC"
    }

### Response
If the `event_id` does not identify an existing event, the endpoint will return a `404 Not Found` response.

`data` will contain the updated Event object on a successful request.

## Delete event

Status: RESTRICTED

Please email support@mobilizeamerica.io to request access to this endpoint.

Delete an in-person event for an organization.

Requires authentication: Yes

### Request
`DELETE /api/v1/organizations/:organization_id/events/:event_id`

### Request Params
* `send_update_notifications`: Defaults to `true`. Set this to `false` to skip notifications. Defaults to sending attendees notifications when the event is deleted.
  *  `DELETE /api/v1/organizations/:organization_id/events/:event_id?send_update_notifications=false`

### Response
On success, the endpoint will return with status code `200 OK`.

# People
## Person object
| Field              | Type      | Description                           |
| ------------------ | --------- | ------------------------------------- |
| `id`               | int       |                                       |
| `created_date`     | int       | Unix timestamp                        |
| `modified_date`    | int       | Unix timestamp                        |
| `given_name`       | string    | First name                            |
| `family_name`      | string    | Last name                             |
| `email_addresses`  | Email[]   | Array of length 1 with an email. (NB: See Email details below for exceptions.) |
| `phone_numbers`    | Phone[]   | Array of length 1 with a phone number |
| `postal_addresses` | Address[] | Array of length 1 with a zip code     |
| `sms_opt_in_status`| enum      | For the current organization; one of `UNSPECIFIED`, `OPT_IN` or `OPT_OUT` |

### Email

| Field     | Type   | Description   |
| --------- | ------ | ------------- |
| `primary` | bool   | Always true   |
| `address` | string | Email address |

Generally a Person will always have exactly one Email, since Person records in our system are uniquely identified by email address. The `email_addresses` list may be empty however if the Person is not affiliated with the requesting Organization, and the Person's affiliated Organization has restrictions on what volunteer data may be shared. This should generally only arise in the context of Attendance list endpoints, and not Person lists, since those should only return affiliated People anyway.

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
| `id`            | int          |                                                                             | If the requesting organization is independent and the event’s organization is coordinated, this is omitted.                                                                                                                                                                                                               |
| `created_date`  | int          | Unix timestamp                                                              |                                                                                                                             |
| `modified_date` | int          | Unix timestamp                                                              |                                                                                                                             |
| `person`        | Person       | Person who attended the event                                               |                                                                                                                             |
| `event`         | Event        | Associated event                                                            | If the requesting organization is independent and the event’s organization is coordinated, all but `event_type` is omitted.  |
| `timeslot`      | Timeslot     | Selected timeslot on event                                                  | If the requesting organization is independent and the event’s organization is coordinated, `id` is omitted.              |
| `sponsor`       | Organization | The promoting organization if it exists, otherwise the event’s organization | If the requesting organization is coordinated and the promoting organization is independent, this is omitted.              |
| `status`        | enum         | `REGISTERED`, `CANCELLED`, or `CONFIRMED`                                   |                                                                                                                             |
| `attended`      | bool         | Whether the person actually attended or not. Will be `null` if not set.     |                                                                                                                             |
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

## Create organization event attendance

Status: RESTRICTED

Please email support@mobilizeamerica.io to request access to this endpoint.

This endpoint creates a new signup for a given person and future event timeslot. If multiple timeslots are provided, an Attendance object will be created for each timeslot. The person is matched and deduplicated by their email address.


Requires authentication: Yes

### Request
`POST /api/v1/organizations/:organization_id/events/:event_id/attendances`

### Request body

| Field             | Type     | Description                                                                                                                                                                                                                                                                                                                                     | Required                       |
| ----------------- | -------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------ |
| person            | AttendancePerson   | Person attendance object. A person is matched and deduplicated by their email address; a record for the person will be created if the `email_address` is not already associated with a person, otherwise the remaining person fields will be updated. | Yes    |
| sms_opt_in_status | string   | One of `OPT_IN`, `UNSPECIFIED`.                                                                                                                                                                                                                                                                                                                 | No (defaults to `UNSPECIFIED`) |
| timeslots         | Timeslot[]    | Array of `timeslot` objects. Must be existing upcoming timeslots for the given Event.                                                                                                                                                                                                                                                               | Yes                            |
| referrer          | Referrer | Referrer object used to add additional tracking to request.                                                           | No                             |

### Person attendance object

| Field            | Type   | Description | Required
| ---------------- | ------ | ----------- | --------
| `given_name`     | string |             | Yes
| `family_name`  | string |             | Yes
| `email_address`  | string |             | Yes
| `phone_number`   | string |             | Yes
| `postal_code`    | string |             | Yes

### Referrer object

| Field          | Type   | Description | Required
| -------------- | ------ | ----------- | --------
| `utm_source`   | string |             | No
| `utm_medium`   | string |             | No
| `utm_campaign` | string |             | No
| `utm_term`     | string |             | No
| `utm_content`  | string |             | No
| `url`          | string |             | No

### Example request body

    {
        "person": {
            "given_name": "myfirstname",
            "family_name": "mylastname",
            "email_address": "email@email.com",
            "phone_number": "1234567890",
            "postal_code": "11111"
        },
        "sms_opt_in_status": "OPT_IN",
        "timeslots": [
            {
                "timeslot_id": 1
            }
        ],
        "referrer": {
            "utm_source": "",
            "utm_medium": "",
            "utm_campaign": "string",
            "utm_term": "string",
            "utm_content": "string",
            "url": "string"
        }
    }

### Response

If the event, organization, or any timeslots do not refer to existing and valid objects, the endpoint will return a 400 Bad Request error with a reference to the invalid fields.

On a successful request, the endpoint will return a 201 Created status code and the newly created Attendance object(s).

### Response Body Example

    {
        "data": [
            {
                "id": 198,
                "attended": null,
                "created_date": 1536588703,
                "modified_date": 1536588703,
                "person": {
                    "id": 57,
                    "created_date": 1536249605,
                    "modified_date": 1536588703,
                    "given_name": "myfirstname",
                    "family_name": "mylastname",
                    "email_addresses": [
                        {
                            "primary": true,
                            "address": "email@email.com"
                        }
                    ],
                    "phone_numbers": [
                        {
                            "primary": true,
                            "number": "1234567890"
                        }
                    ],
                    "postal_addresses": [
                        {
                            "primary": true,
                            "postal_code": "11111"
                        }
                    ],
                    "sms_opt_in_status": "OPT_IN"
                },
                "event": {
                    "id": 345,
                    "description": "abacde",
                    "timezone": "America/Chicago",
                    "title": "abcdef",
                    "summary": "",
                    "featured_image_url": "",
                    "sponsor": {
                        "id": 15,
                        "name": "My campaign",
                        "slug": "campaign123",
                        "is_coordinated": true,
                        "is_independent": false,
                        "is_primary_campaign": false,
                        "state": "",
                        "district": "",
                        "candidate_name": "",
                        "race_type": null,
                        "event_feed_url": "events.campaign.com",
                        "created_date": 1517329224,
                        "modified_date": 1536160166
                    },
                    "timeslots": null,
                    "location": {
                        ...
                    },
                    "event_type": "TRAINING",
                    "created_date": 1535987085,
                    "modified_date": 1535987575,
                    "browser_url": "events.campaign.com/event/345/",
                    "high_priority": null,
                    "contact": null
                },
                "timeslot": {
                    "id": 1,
                    "start_date": 1536679800,
                    "end_date": 1536690600
                },
                "sponsor": {
                    "id": 15,
                    "name": "Campaign",
                    "slug": "campaign123",
                    "is_coordinated": true,
                    "is_independent": false,
                    "is_primary_campaign": false,
                    "state": "",
                    "district": "",
                    "candidate_name": "",
                    "race_type": null,
                    "event_feed_url": "events.campaign.com",
                    "created_date": 1517329224,
                    "modified_date": 1536160166
                },
                "status": "REGISTERED",
                "referrer": {
                    "utm_source": null,
                    "utm_medium": null,
                    "utm_campaign": "string",
                    "utm_term": "string",
                    "utm_content": "string",
                    "url": "string"
                }
            }
        ],
        "error": null
    }


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

# Affiliation


## Create organization affiliations

Status: RESTRICTED

Please email support@mobilizeamerica.io to request access to this endpoint.

This endpoint creates a new affiliation between the given person and organization, or updates a person’s contact information if an affiliation already exists. The person is matched and deduplicated by their email address.

Requires authentication: Yes

### Request
`POST /api/v1/organizations/:organization_id/affiliations`

### Request body
| Field             | Type   | Description                                       | Required |
| ----------------- | ------ | ------------------------------------------------- | -------- |
| given_name        | string | The first name of the person.                     | Yes      |
| family_name       | string | The last name of the person.                      | Yes      |
| email_address     | string | The email address of the person.                  | Yes      |
| phone_number      | string | The phone number of the person.                   | Yes      |
| address_line_1    | string | The first address line of the person’s location.  | No       |
| address_line_2    | string | The second address line of the person’s location. | No       |
| locality          | string | The city of the person’s location.                | No       |
| region            | string | The U.S. state of the person’s location.          | No       |
| postal_code       | string | The zipcode of the person’s location.             | Yes      |
| sms_opt_in_status | string | One of `OPT_IN`, `UNSPECIFIED`.                   | Yes      |


### Request body example

    {
        "person": {
            "given_name": "myfirstname",
            "family_name": "mylastname",
            "email_address": "email@email.com",
            "phone_number": "1234567890",
            "address_line_1": "123 Main St.",
            "address_line_2": "",
            "locality": "New York",
            "region": "NY",
            "postal_code": "10001"
        },
        "sms_opt_in_status": "OPT_IN"
    }

### Response

If any required fields are missing or contain invalid values, the endpoint will return a 400 Bad Request with references to the invalid fields.

On a successful request, the endpoint will return a 201 Created status code if the person record was created, a 200 No Content result if the person record was updated, and the affected Affiliation object.

# Changelog

**2018-10-23**
- Add `visibility` to Event object and as a request param for organization events endpoints

**2018-10-08**
- Omit `id` from attendance GET endpoints when the requesting organization is independent and the event's organization is coordinated

**2018-10-01**
- Add request param `send_update_notifications` to Event update and delete

**2018-09-13**
- Add endpoint to create new event attendance
- Add endpoint to create new affiliation between a person and an organization

**2018-09-11**
- Expanded list of available event types: CANVASS, PHONE_BANK, TEXT_BANK, MEETING, COMMUNITY, FUNDRAISER, MEET_GREET, HOUSE_PARTY, VOTER_REG, TRAINING, FRIEND_TO_FRIEND_OUTREACH, OTHER

**2018-09-05**
- Add contact information to Event object, returned only for authenticated requests

**2018-09-04**
- Add `congressional_district`, `state_leg_district`, and `state_senate_district` to Event Location object.

**2018-09-03**
- Add endpoint to create new event
- Add endpoint to update existing event
- Add endpoint to delete event

**2018-08-23**
- Add filtering of events by zipcode and optional maximum distance away from that zipcode

**2018-08-09**
- Add ability to pass "now" in place of a Unix timestamp in timestamp comparison filters

**2018-07-11**
- Add `timeslot_end` filter to public events and organization events endpoints

**2018-06-14**
- Add `high_priority` to Event object

**2018-06-11**
- Add `sms_opt_in_status` to Person object

**2018-05-23**
- Add `attended` to Attendance object

**2018-05-08**

- Add endpoint for deleted events
- Add endpoint for deleted events for an organization and its promoted organizations
- Add filtering by comparisons for timeslot start dates on events

**2018-05-03**

- Add endpoint to fetch promoted organizations
- Add endpoint to fetch affiliated people
- Add endpoint to fetch attendances for an organization
- Add endpoint to fetch attendances for a person
- Added `url` to Referrer object
