# MobilizeAmerica API

Please see the [Changelog](#changelog) below for changes, and file any issues [here](https://github.com/mobilizeamerica/api/issues)

To stay updated on new releases or iterations, join the email list [here](https://groups.google.com/forum/#!forum/mobilizeamerica-api-users)

# Table of Contents

- [MobilizeAmerica API](#mobilizeamerica-api)
- [Table of Contents](#table-of-contents)
- [Overview](#overview)
  - [API entry point](#api-entry-point)
  - [Format](#format)
  - [Authentication](#authentication)
  - [Errors](#errors)
  - [Paging](#paging)
    - [`results_limited_to`](#results_limited_to)
    - [`cursor`](#cursor)
  - [Comparison filters](#comparison-filters)
  - [OSDI compliance](#osdi-compliance)
  - [Rate limits](#rate-limits)
- [Organizations](#organizations)
  - [Organization object](#organization-object)
  - [List all organizations](#list-all-organizations)
    - [Request](#request)
    - [Request params](#request-params)
    - [Response](#response)
  - [List all the organizations promoted by an organization](#list-all-the-organizations-promoted-by-an-organization)
    - [Request](#request-1)
    - [Request params](#request-params-1)
    - [Response](#response-1)
- [Events](#events)
  - [Event object](#event-object)
    - [Timeslot](#timeslot)
    - [Location](#location)
    - [Contact](#contact)
    - [Tag](#tag)
    - [EventCampaign](#eventcampaign)
    - [Deleted Event](#deleted-event)
  - [**[DEPRECATED]** List all public events](#deprecated-list-all-public-events)
    - [Request](#request-2)
    - [Request params](#request-params-2)
    - [Response](#response-2)
  - [Get a public event](#get-a-public-event)
    - [Request](#request-3)
    - [Request params](#request-params-3)
    - [Response](#response-3)
  - [List organization events](#list-organization-events)
    - [Request](#request-4)
    - [Request params](#request-params-4)
    - [Response](#response-4)
  - [Get an organization event](#get-an-organization-event)
    - [Request](#request-5)
    - [Request params](#request-params-5)
    - [Response](#response-5)
  - [**[DEPRECATED]** List deleted public events](#deprecated-list-deleted-public-events)
    - [Request](#request-6)
    - [Request params](#request-params-6)
    - [Response](#response-6)
  - [List deleted organization events](#list-deleted-organization-events)
    - [Request](#request-7)
    - [Request params](#request-params-7)
    - [Response](#response-7)
  - [Create event](#create-event)
    - [Request](#request-8)
    - [Request body](#request-body)
    - [Request body example](#request-body-example)
    - [Response](#response-8)
  - [Update event](#update-event)
    - [Request](#request-9)
    - [Request params](#request-params-8)
    - [Request body](#request-body-1)
    - [Request body example](#request-body-example-1)
    - [Response](#response-9)
  - [Delete event](#delete-event)
    - [Request](#request-10)
    - [Request params](#request-params-9)
    - [Response](#response-10)
- [People](#people)
  - [Person object](#person-object)
    - [Email](#email)
    - [Phone](#phone)
    - [Address](#address)
  - [List organization people](#list-organization-people)
    - [Request](#request-11)
    - [Request params](#request-params-10)
    - [Response](#response-11)
- [Attendances](#attendances)
  - [Attendance object](#attendance-object)
    - [Referrer](#referrer)
    - [CustomSignupFieldValue](#customsignupfieldvalue)
  - [Get an organization attendance](#get-an-organization-attendance)
    - [Request](#request-12)
    - [Response](#response-12)
  - [List organization attendances](#list-organization-attendances)
    - [Request](#request-13)
    - [Request params](#request-params-11)
    - [Response](#response-13)
  - [List organization attendances for an event](#list-organization-attendances-for-an-event)
    - [Request](#request-14)
    - [Response](#response-14)
  - [Create organization event attendance](#create-organization-event-attendance)
    - [Request](#request-15)
    - [Request body](#request-body-2)
    - [Person attendance object](#person-attendance-object)
    - [Referrer object](#referrer-object)
    - [Request body example](#request-body-example-2)
    - [Response](#response-15)
    - [Response body example](#response-body-example)
- [Affiliation](#affiliation)
  - [Create organization affiliations](#create-organization-affiliations)
    - [Request](#request-16)
    - [Request body](#request-body-3)
    - [Request body example](#request-body-example-3)
    - [Response](#response-16)
- [Images](#images)
  - [Upload images](#upload-images)
    - [Request](#request-17)
    - [Request body](#request-body-4)
    - [Response](#response-17)
- [Enums](#enums)
  - [Get enums](#get-enums)
    - [Request](#request-18)
    - [Response](#response-18)
- [Staging and Development](#staging-and-development)
- [Changelog](#changelog)

# Overview
## API entry point

All requests occur through `https://api.mobilize.us/v1`.

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

Here’s an example error response for `/v1/events?organization_id=hello`:


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

### `results_limited_to`

Some endpoints will hit our search backend when filtered on many parameters at
once (like our our [List Organization Events](#list-organization-events)
endpoint), which limits the total number of objects we can return for a given
search. If that happens, we will return a `results_limited_to` field in the
response which will be an integer with the total number of objects that the
endpoint will return, usually 1000. This number may not correspond to the
`count` field, so we advise consumers consult this field to determine whether
their request will be limited.

### `cursor`
We default to using cursor based pagination for performance reasons. This uses the `cursor` query param, and will populate in the `next` and `previous` links. This still respects `per_page`. The data returned should be the same as if you increment the `page` query param.

You can still use the `page` param to select a starting page, and cursor pagination will start from there with the `next` and `previous` links.

## Comparison filters

Some endpoints will allow for filtering by comparing with a field within an object. In those cases, the format will be `field_name=cmp_####`. For example, to filter out events before Jan 1 2018 GMT, you can include `timeslot_start=gte_1514764800`. Multiple may also be used, e.g. `timeslot_start=gte_1514764800&timeslot_start=lt_1515110400`. The comparison operators are ≥ `gte`, > `gt`, ≤ `lte`, < `lt`. More may be added in the future.

For timestamp comparisons, the string "now" can be passed in place of a Unix timestamp to have the server compute the current time, e.g., `timeslot_start=gte_now`.

## OSDI compliance

OSDI is an exciting and important attempt to bring interoperability to the progressive data ecosystem, a cause we heartily support. However, there are a number of data model and format constraints of OSDI that made it a less-than-perfect fit for us and our consumers. While we have opted not to follow strict OSDI formats for our API responses at this stage, we have made our best effort to align field names and structure where possible. For example, many of the Event fields map directly onto OSDI field names. We hope this will make the job of anyone attempting to build an OSDI layer on top of our API easier.

## Rate limits

We currently rate-limit API requests per origin IP to 15 requests per second for GET (read) endpoints and 5 requests per second for POST/PUT (write) endpoints. We strongly encourage you to keep requests well under these limits, as we may adjust the limits at short notice. Rate limited requests will receive an HTTP 429 response code. We are happy to work with developers who are concerned about their use-case hitting limits, just reach out to support@mobilize.us.

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
| `org_type`            | string | One of `CAMPAIGN`, `STATE_PARTY`, `COORDINATED`, `INDEPENDENT` New options may be added.                                                                                                      |

## List all organizations

Status: LIVE

Return all active organizations on the platform. This endpoint is publicly accessible.

Requires authentication: No

### Request
`GET /v1/organizations`

### Request params

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
          "event_feed_url": "https://www.mobilize.us/example/",
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
`GET /v1/organizations/:organization_id/promoted_organizations`

### Request params

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
          "event_feed_url": "https://www.mobilize.us/example/",
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
| `description`        | string       | Long-form HTML description of the event                                                                                                                 |
| `featured_image_url` | string       | Path to the image for the event                                                                                                                         |
| `high_priority`      | bool         | Whether the event is marked high priority for the provided organization                                                                                 |
| `sponsor`            | Organization | The owning Organization, with the fields defined above                                                                                                  |
| `timeslots`          | Timeslot[]   | Array of past and future timeslots                                                                                                                      |
| `location`           | Location     | The event location, or `null` if the event has no location                                                                                              |
| `timezone`           | string       | A timezone database string for the event, e.g., `America/New_York`.                                                                                     |
| `event_type`         | enum         | The type of the event, one of: `CANVASS`, `PHONE_BANK`, `TEXT_BANK`, `MEETING`, `COMMUNITY`, `FUNDRAISER`, `MEET_GREET`, `HOUSE_PARTY`, `VOTER_REG`, `TRAINING`, `FRIEND_TO_FRIEND_OUTREACH`, `DEBATE_WATCH_PARTY`, `ADVOCACY_CALL`, `RALLY`, `TOWN_HALL`, `OFFICE_OPENING`, `BARNSTORM`, `SOLIDARITY_EVENT`, `COMMUNITY_CANVASS`, `SIGNATURE_GATHERING`, `CARPOOL`, `WORKSHOP`, `PETITION`, `AUTOMATED_PHONE_BANK`, `LETTER_WRITING`, `LITERATURE_DROP_OFF`, `VISIBILITY_EVENT`, `OTHER`. This list may expand. |
| `browser_url`        | string       | Canonical URL of the event                                                                                                                              |
| `created_date`       | int          | Unix timestamp                                                                                                                                          |
| `modified_date`      | int          | Unix timestamp                                                                                                                                          |
| `visibility`         | enum         | The visibility of the event, one of: `PUBLIC`, `PRIVATE`.                                                                                               |
| `address_visibility` | enum         | The visibility of the event's address (which may be different from the visibility of the event itself). One of `PUBLIC`, `PRIVATE`.                     |
| `created_by_volunteer_host`      | bool          | Whether the event was created by a volunteer host using our distributed organizing tool or not                                                                                                                                          |
| `is_virtual`         | bool         | Whether the event is virtual or not                                                                                                                     |
| `virtual_action_url` | string       | The url the event redirects to if it's an unshifted virtual event. Otherwise, `null`.                                                                   |
| `contact`            | Contact      | The contact information for the event |
| `accessibility_status`           | enum          | The degree of compliance with the Americans with Disabilities Act. One of: `ACCESSIBLE`, `NOT_ACCESSIBLE`, `NOT_SURE`, or `null`           |
| `accessibility_notes`| string       | Additional details about accessibility status and accomodations at the venue                                                                            |
| `tags`               | Tag[]        | Array of associated tags                                                                                                                                |
| `approval_status`    | enum         | For a distributed organizing event, denotes where it is in the approval process. For other events, will always be `APPROVED`. One of: `APPROVED`, `NEEDS_APPROVAL`, `REJECTED`, `NEEDS_HOST_VERIFICATION` |
| `event_campaign`               | EventCampaign        | The associated distributed organizing event campaign, if applicable. Only exposed for authenticated [list organization events](#list-organization-events) or [get organization event](#get-an-organization-event) requests, for events owned by the authenticated user's organization. `null` otherwise. |
| `instructions`            | string      | Private instructions sent to attendees of this event after signing up. Only exposed for authenticated [list organization events](#list-organization-events) or [get organization event](#get-an-organization-event) requests, for events owned by the authenticated user's organization. `null` otherwise. |

### Timeslot

When [creating events](#create-event), note that only `start_date`, `end_date`, and optionally `max_attendees` should be provided.

| Field           | Type          | Description    |
| --------------- | ----          | -------------- |
| `id`            | int           |                |
| `start_date`    | int           | Unix timestamp |
| `end_date`      | int           | Unix timestamp |
| `max_attendees` | Optional[int] | Max # of people who can sign up for this timeslot (a.k.a. capacity). Send null for no maximum. |
| `is_full`       | bool          | Whether the timeslot is full |
| `instructions`  | string        | Private instructions sent to attendees of this timeslot after signing up. Only exposed for authenticated [list organization events](#list-organization-events) requests, for events owned by the authenticated user's organization. `null` otherwise. |


### Location

| Field                    | Type     | Description                                                                                                |
| ------------------------ | -------- | ---------------------------------------------------------------------------------------------------------- |
| `venue`                  | string   | The name of the location, e.g., “Campaign HQ” or “Starbucks”. If the location is private, it will be the string `This event’s address is private. Sign up for more details` |
| `address_lines`          | string[] | The lines of the address. Should always have exactly two values in our system, which may be empty strings. If the location is private, the first line will be the string `This event’s address is private. Sign up for more details` |
| `locality`               | string   | The city                                                                                                   |
| `region`                 | string   | The two-character state code                                                                               |
| `country`                 | string   | An [ISO-3166-1 alpha-2 country code](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2). Note that U.S. territories and commonwealths have their own country codes; e.g., Puerto Rico is `PR`. For create and update requests, the field is optional and defaults to `US`. Currently, only `US` is accepted.  |
| `postal_code`            | string   | The zipcode                                                                                                |
| `location`               | object   | The geocoded location, or `null` if geocoding failed.                                                      |
| `location.latitude`      | float    |                                                                                                            |
| `location.longitude`     | float    |                                                                                                            |
| `congressional_district` | string   | The Congressional district, or `null` if geocoding failed or no street address was provided                |
| `state_leg_district`     | string   | The State Lower House district, or `null` if geocoding failed or no street address was provided            |
| `state_senate_district`  | string   | The State Upper House/Senate district, or `null` if geocoding failed or no street address was provided     |


### Contact
The Contact object will return `null` for unauthenticated requests.
Note that the name, email address, and phone number may differ from the contact information of the owner if they have been set specifically for the event.

When [creating events](#create-event), note that `owner_user_id` should not be supplied.


| Field            | Type    | Description                                     |
| ---------------- | ------- | ----------------------------------------------- |
| `name`           | string  | The name of the contact for the event.          |
| `email_adddress` | string  | The email address of the contact for the event. |
| `phone_number`   | string  | The phone number of the contact for the event.  |
| `owner_user_id`  | int     | The user_id of the user who owns the event.     |


### Tag

| Field        | Type   | Description          |
| ------------ | ------ | -------------------- |
| `id`         | int    |                      |
| `name`       | string | Text name of the tag |

### EventCampaign

| Field        | Type   | Description          |
| ------------ | ------ | -------------------- |
| `id`         | int    |                      |
| `slug`       | string | The public-facing slug of the event campaign. |
| `event_create_page_url` | string | The URL of the public-facing event creation page for the event campaign. |


### Deleted Event
| Field          | Type | Description    |
| -------------- | ---- | -------------- |
| `id`           | int  |                |
| `deleted_date` | int  | Unix timestamp |

## **[DEPRECATED]** List all public events

**DEPRECATION NOTICE: This endpoint is no longer supported, and will soon function as
an alias for [List Organization Events](#list-organization-events) with
`organization_id` = `1`. See our [Deprecation
Notice](https://github.com/mobilizeamerica/api/pull/64) for details.**


Status: DEPRECATED

Fetch all public events on the platform. To list an organization’s private events, send an
authenticated request to the [organization events list](#list-organization-events) endpoint instead.

Requires authentication: No*

If you have privileged data access (e.g. contact details and private location information) to the
returned events, you'll need to send an authenticated request to see that data.

### Request
`GET /v1/events`

### Request params

- `organization_id`: One or more Organization IDs to filter to. If multiple, should be supplied as multiple query params, e.g., `organization_id=1&organization_id=2`, etc.
- `updated_since`: Unix timestamp to filter by Events’ `modified_date`
- `timeslot_start`: Comparison to filter by Events' Timeslots' start date. Will only return Timeslots on those Events that meet the filter conditions
- `timeslot_end`: Comparison to filter by Events' Timeslots' end date. Will only return Timeslots on those Events that meet the filter conditions
- `zipcode`: Zipcode to filter by Events' Locations' postal code. If present, will return Events sorted by distance from zipcode. To improve performance, query results are limited to a maximum of 1000.
- `max_dist`: Maximum distance (in miles) to filter by Events' Locations' distance from provided zipcode.
- `exclude_full`: Boolean; whether to filter out full Timeslots (and Events, if all of an Event's Timeslots are full), e.g. `exclude_full=true`
- `is_virtual`: Optional boolean, e.g. `is_virtual=false` will return only in-person events, while `is_virtual=true` will return only virtual events. If excluded, return virtual and in-person events.
- `event_types`: One or more event types to filter to (see possible values in `event_type` on the [Event object](#event-object)). If multiple, should be supplied as multiple query params, e.g. `event_types=CANVASS&event_types=PHONE_BANK`.
- `tag_id`: One or more Tag IDs to filter to. If multiple, should be supplied as multiple query params, e.g., `tag_id=1&tag_id=2`, etc.

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
          "featured_image_url": "",
          "high_priority": null,
          "sponsor": {
            Organization object
          },
          "timeslots": [
            {
              "id": 1,
              "start_date": 2,
              "end_date": 3,
              "instructions": null
            },
            {
              "id": 2,
              "start_date": 3,
              "end_date": 4,
              "instructions": null
            },
          ],
          "is_virtual": false,
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
          "browser_url": "https://www.mobilize.us/event/1/"
          "contact": {
            "name": "",
            "email_address": "replyto@thisemail.org",
            "phone_number": "1234567890"
          },
          "event_campaign": null,
          "instructions": null
        },
        ...
      ]
    }

## Get a public event

Status: LIVE

Fetch a public event on the platform by id. This endpoint will 404 if a private event is requested -
to fetch a private event, use the [organization event endpoint](#get-an-organization-event) instead.

Requires authentication: No*

If you have privileged data access (e.g. contact details and private location information) to the
returned event, you'll need to send an authenticated request to see that data.

### Request

`GET /v1/events/:event_id`

### Request params

None

### Response

`data` is the returned Event object.

    {
      ...,
      "data": {
        "id": 1,
        "description": "example",
        "timezone": "America/New_York",
        "title": "Example",
        "featured_image_url": "",
        "high_priority": null,
        "sponsor": {
          Organization object
        },
        "timeslots": [
          {
            "id": 1,
            "start_date": 2,
            "end_date": 3,
            "instructions": null
          },
          {
            "id": 2,
            "start_date": 3,
            "end_date": 4,
            "instructions": null
          },
        ],
        "is_virtual": false,
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
        "browser_url": "https://www.mobilize.us/event/1/"
        "contact": {
          "name": "",
          "email_address": "replyto@thisemail.org",
          "phone_number": "1234567890"
        },
        "event_campaign": null,
        "instructions": null
      },
    }

## List organization events

Status: LIVE

Fetch all events for an organization. This includes both events owned by the organization
(as indicated by the `organization` field on the event object) and events of other organizations
promoted by this specified organization. By default, this endpoint will return only public events.

Requires authentication: No*

While authentication is not required for this endpoint, if it’s not provided then private events
will be excluded from the results. Additionally, if you have privileged data access (e.g. contact
details, private location information, and event campaign details) to the returned events, you’ll
need to send an authenticated request to see that data.

### Request
`GET /v1/organizations/:organization_id/events`

### Request params

- `updated_since`: Unix timestamp to filter by Events’ `modified_date`
- `timeslot_start`: Comparison to filter by Events' Timeslots' start date. Will only return Timeslots on those Events that meet the filter conditions
- `timeslot_end`: Comparison to filter by Events' Timeslots' end date. Will only return Timeslots on those Events that meet the filter conditions
- `visibility`: Type of event visibility to filter by; either `PUBLIC` or `PRIVATE`. Private events will only be returned if `visibility=PRIVATE` is specified, the calling user is authenticated, and the user has permission to view the given organization's private events in the dashboard. If `visibility=PRIVATE` is specified and the calling user does not have permission, a 403 error is returned. Defaults to public events only. Note also that both can be specified, ie `visibility=PRIVATE&visibility=PUBLIC`.
- `organization_id`: One or more Organization IDs, which will filter the events down to who the owning organization is (since normally results would include events that are owned, co-owned, OR promoted by the organization in the URL path).
- `zipcode`*: Zipcode to filter by Events' Locations' postal code. If present, will return Events sorted by distance from zipcode. To improve performance, query results are limited to a maximum of 1000.
- `max_dist`*: Maximum distance (in miles) to filter by Events' Locations' distance from provided zipcode.
- `exclude_full`*: Boolean; whether to filter out full Timeslots (and Events, if all of an Event's Timeslots are full), e.g. `exclude_full=true`
- `is_virtual`: Optional boolean, e.g. `is_virtual=false` will return only in-person events, while `is_virtual=true` will return only virtual events. If excluded, return virtual and in-person events.
- `high_priority_only`: Optional boolean, e.g. `high_priority_only=true` will return only high priority ("pinned") events. If excluded or false, return high priority and non-high-priority events.
- `event_types`: One or more event types to filter to (see possible values in `event_type` on the [Event object](#event-object)). If multiple, should be supplied as multiple query params, e.g. `event_types=CANVASS&event_types=PHONE_BANK`.
- `tag_id`*: One or more Tag IDs to filter to. If multiple, should be supplied as multiple query params, e.g., `tag_id=1&tag_id=2`, etc.
- `event_campaign_id`: Optional EventCampaign ID to filter to.
- `approval_status`: One or more approval statuses to filter to (see possible values in `approval_status` on the [Event object](#event-object)). Defaults to approved events only. Non-approved events will only be returned if explicitly specified, the calling user is authenticated, and the user has permission to view the given organization's non-approved events in the dashboard. For example, to request both approved and needs approval events, specify `approval_status=APPROVED&approval_status=NEEDS_APPROVAL`.
- `state`: The two-letter state abbreviation code to filter Events' Locations by. If present, will only return Events with their location set to this state.

*&nbsp;The presence of any request params marked with a * will trigger our search
backend, which will limit the total results to the value provided in the
[`results_limited_to`](#results_limited_to) field

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
          "featured_image_url": "",
          "high_priority": true,
          "sponsor": {
            Organization object
          },
          "timeslots": [
            {
              "id": 1,
              "start_date": 2,
              "end_date": 3,
              "instructions": "",
            },
            {
              "id": 2,
              "start_date": 3,
              "end_date": 4,
              "instructions": "Some detailed instructions for the second timeslot",
            }
          ],
          "is_virtual": false,
          "location": {
            "venue": "",
            "address_lines": [
              "204 E 13th St",
              ""
            ],
            "locality": "New York",
            "region": "NY",
            "country": "US",
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
          "browser_url": "https://www.mobilize.us/event/1/"
          "contact": {
            "name": "",
            "email_address": "replyto@thisemail.org",
            "phone_number": "1234567890"
          },
          "event_campaign": {
            EventCampaign object
          },
          "instructions": "Bring a hat"
        },
        ...
      ]
    }

## Get an organization event

Status: LIVE

Fetch a single event for an organization. This can be an event owned by the organization
(as indicated by the `organization` field on the event object) or an event of another
organization promoted by the specified organization. If the event requested is neither
owned nor promoted by this organization, the endpoint will return a `404 NOT FOUND`.

Requires authentication: No*

If you have privileged data access (e.g. contact details, private location information, and event
campaign details) to the returned event, you'll need to send an authenticated request to see that
data.

### Request
`GET /v1/organizations/:organization_id/events/:event_id`

### Request params

None

### Response
`data` is the Event object requested.

    {
      ...,
      "data": {
        "id": 1,
        "description": "example",
        "timezone": "America/New_York",
        "title": "Example",
        "featured_image_url": "",
        "high_priority": true,
        "sponsor": {
          Organization object
        },
        "timeslots": [
          {
            "id": 1,
            "start_date": 2,
            "end_date": 3,
            "instructions": ""
          },
          {
            "id": 2,
            "start_date": 3,
            "end_date": 4,
            "instructions": "Some detailed instructions for the second timeslot"
          }
        ],
        "is_virtual": false,
        "location": {
          "venue": "",
          "address_lines": [
            "204 E 13th St",
            ""
          ],
          "locality": "New York",
          "region": "NY",
          "country": "US",
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
        "browser_url": "https://www.mobilize.us/event/1/"
        "contact": {
          "name": "",
          "email_address": "replyto@thisemail.org",
          "phone_number": "1234567890"
        },
        "event_campaign": {
          EventCampaign object
        },
        "instructions": "Bring a hat"
      }
    }

## **[DEPRECATED]** List deleted public events

**DEPRECATION NOTICE: This endpoint is no longer supported, and now functions
as an alias for [List Deleted Organization
Events](#list-deleted-organization-events) with `organization_id` = `1`. See
our [Deprecation Notice](https://github.com/mobilizeamerica/api/pull/64) for
details.**

Status: DEPRECATED

Fetch deleted public events on the platform.

Requires authentication: No

### Request
`GET /v1/events/deleted`

### Request params

- `organization_id`: One or more Organization IDs to filter to. If multiple, should be supplied as multiple query params, e.g., `organization_id=1&organization_id=2`, etc.
- `updated_since`: Unix timestamp to filter by Events’ `modified_date`

### Response
`data` is an array of Deleted Event objects.

## List deleted organization events

Status: LIVE

Fetch deleted events for an organization. This includes both events owned by the organization (as indicated by the `organization` field on the event object) and events of other organizations promoted by this specified organization.

Requires authentication: No*

While authentication is not required for this endpoint, if it’s not provided then private events will be excluded from the results.

### Request
`GET /v1/organizations/:organization_id/events/deleted`

### Request params

- `updated_since`: Unix timestamp to filter by Events’ `modified_date`
- `visibility`: Type of event visibility to filter by; either `PUBLIC` or `PRIVATE`. Private events will only be returned if `visibility=PRIVATE` is specified, the calling user is authenticated, and the user has permission to view the given organization's private events in the dashboard. If `visibility=PRIVATE` is specified and the calling user does not have permission, no events will be returned. Defaults to public events only. Note also that both can be specified, ie `visibility=PRIVATE&visibility=PUBLIC`.

### Response
`data` is an array of Deleted Event objects.

## Create event

Status: RESTRICTED

Please email support@mobilize.us to request access to this endpoint.

Create a new in-person Event for an given organization.

Requires authentication: Yes

### Request
`POST /v1/organizations/:organization_id/events`

### Request body

| Field                | Type         | Description                                                                                                                                             | Required |
| -------------------- | ------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------- | --------
| `title`              | string       | The public name of the event                                                                                                                                                                            | Yes
| `description`        | string       | Long-form HTML description of the event                                                                                                                                                                 | Yes
| `timeslots`          | Timeslot[]   | Array of future [Timeslots](#timeslot), containing `start_date` and `end_date` fields, and optionally `max_attendees` and `instructions` as well. Timeslots must be in Unix time.                                                                                                                                     | Yes
| `location`           | Location     | The event location. Required if `is_virtual` is `false` or unset. `postal_code` is a required field in the `location` object; all other `location` fields are optional. | No
| `is_virtual`         | bool         | Whether the event is virtual or not. Defaults to `false`. Cannot be updated after event creation. | No
| `timezone`           | string       | A timezone database string for the event, one of: `America/New_York`, `Pacific/Honolulu`, `America/Los_Angeles`, `America/Denver`, `America/Phoenix`, `America/Chicago`.                                | Yes
| `event_type`         | string       | The type of the event, one of: `CANVASS`, `PHONE_BANK`, `TEXT_BANK`, `MEETING`, `COMMUNITY`, `FUNDRAISER`, `OTHER`, `MEET_GREET`, `HOUSE_PARTY`, `VOTER_REG`, `TRAINING`, `FRIEND_TO_FRIEND_OUTREACH`, `DEBATE_WATCH_PARTY`, `RALLY`, `TOWN_HALL`, `OFFICE_OPENING`, `BARNSTORM`, `SOLIDARITY_EVENT`, `COMMUNITY_CANVASS`, `SIGNATURE_GATHERING`, `CARPOOL`, `WORKSHOP`, `PETITION`, `AUTOMATED_PHONE_BANK`, `LETTER_WRITING`, `LITERATURE_DROP_OFF`, `VISIBILITY_EVENT`, `PLEDGE`, `INTEREST_FORM`, `DONATION_CAMPAIGN`, `SOCIAL_MEDIA_CAMPAIGN`, `POSTCARD_WRITING`, `GROUP`, `VOLUNTEER_SHIFT`. Note that creating events with event type `ADVOCACY_CALL` is not currently supported in the API. | Yes
| `visibility`         | string       | The visibility of the event, one of: `PUBLIC`, `PRIVATE`. | Yes
| `contact`            | Contact      | A [Contact object](#contact) containing contact info for the event. | Yes
| `accessibility_status` | string     | The level of compliance with the [Americans with Disabilities Act](https://www.access-board.gov/guidelines-and-standards/buildings-and-sites/about-the-ada-standards/guide-to-the-ada-standards/single-file-version) for the event venue, one of: `ACCESSIBLE`, `NOT_ACCESSIBLE`, `NOT_SURE`. If you set this to `ACCESSIBLE`, you are responsible for ensuring that your venue meets ADA standards. | No
| `accessibility_notes`  | string     | Notes with additional information about accessibility at the event location, including the availability of ramps and wheelchair-accessible restrooms, the height of door thresholds, the number of stairs, and the nature of any parking or seating arrangements. This is helpful even for venues that are not fully ADA accessible.| No
| `featured_image_url`   | string      | The Mobilize-hosted image URL for the event. Must be generated using the [Upload images](#upload-images) endpoint. | No
| `tag_ids`              | int[]       | Array of ids of [Tags](#Tags) to apply to the event      | No
| `instructions`         | string      | Private instructions sent to attendees after signing up. | No

### Request body example
```json
    {
        "title": "Example",
        "description": "example",
        "timezone": "America/New_York",
        "timeslots": [
            {
              "start_date": 1576774800,
              "end_date": 1576782000,
            },
            {
              "start_date": 1576861200,
              "end_date": 1576868400,
              "instructions": "Some instructions for just this timeslot"
            }
        ],
        "location": {
            "venue": "Starbucks",
            "address_lines": [
                "315 7th Ave",
                ""
            ],
            "locality": "New York",
            "region": "NY",
            "postal_code": "10003"
        },
        "event_type": "CANVASS",
        "visibility": "PUBLIC",
        "contact": {
            "email_address": "replyto@thisemail.com"
        },
        "accessibility_status": "ACCESSIBLE",
        "accessibility_notes": "There is a wheelchair ramp at the southern entrance for the staging area. We have two vans with wheelchair lifts.",
        "featured_image_url": "https://mobilize-staging.imgix.net/uploads/event/test_20191203233112123932.jpg",
        "tag_ids": [3, 34],
        "instructions": "Bring a hat"
    }
```

### Response
`data` will contain the newly created Event object.

## Update event

Status: RESTRICTED

Please email support@mobilize.us to request access to this endpoint.

Update an in-person event for an organization. `event_id` refers to the `id` field in the Event object.

Note that all editable fields must be specified, otherwise they will be removed (e.g. an existing `timeslot`) or set to null (e.g. the `featured_image_url` field).

Requires authentication: Yes

### Request
`PUT /v1/organizations/:organization_id/events/:event_id`

### Request params
* `send_update_notifications`: Defaults to `true`. Set this to `false` to skip notifications. Defaults to sending email notifications to attendees when `location` or `timeslots` are updated.
  * `PUT /v1/organizations/:organization_id/events/:event_id?send_update_notifications=false`

### Request body
| Field                | Type         | Description                                                                                                                                             | Required |
| -------------------- | ------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------- | --------
| `title`              | string       | The public name of the event                                                                                                                                                                            | Yes
| `description`        | string       | Long-form HTML description of the event                                                                                                                                                                 | Yes
| `timeslots`          | Timeslot[]   | Array of future timeslots. Timeslots must be a valid Unix timestamp. Any existing, upcoming timeslots that are not present in the `timeslots` array will be deleted. Existing, upcoming timeslots may be updated by including the `id` field in the `timeslot` object. Past timeslots will not be modified by this endpoint.                                                                                                                                     | Yes
| `location`           | Location     | The event location. Required if `is_virtual` is `false` or unset. `postal_code` is a required field in the `location` object; all other `location` fields are optional. | No
| `is_virtual`         | bool         | Whether the event is virtual or not. Defaults to `false`. Cannot be updated after event creation. | No
| `timezone`           | string       | A timezone database string for the event, one of: `America/New_York`, `Pacific/Honolulu`, `America/Los_Angeles`, `America/Denver`, `America/Phoenix`, `America/Chicago`.                                | Yes
| `event_type`         | string       | The type of the event, one of: `CANVASS`, `PHONE_BANK`, `TEXT_BANK`, `MEETING`, `COMMUNITY`, `FUNDRAISER`, `MEET_GREET`, `HOUSE_PARTY`, `VOTER_REG`, `TRAINING`, `FRIEND_TO_FRIEND_OUTREACH`, `DEBATE_WATCH_PARTY`, `RALLY`, `TOWN_HALL`, `OFFICE_OPENING`, `BARNSTORM`, `SOLIDARITY_EVENT`, `COMMUNITY_CANVASS`, `SIGNATURE_GATHERING`, `CARPOOL`, `WORKSHOP`, `PETITION`, `AUTOMATED_PHONE_BANK`, `LETTER_WRITING`, `LITERATURE_DROP_OFF`, `VISIBILITY_EVENT`, `OTHER`. Note that updating events to event type `ADVOCACY_CALL` is not currently supported in the API.  | Yes
| `contact`            | Contact      | A [Contact object](#contact) containing contact info for the event. | Yes
| `visibility`         | string       | The visibility of the event, one of: `PUBLIC`, `PRIVATE`.   | Yes
| `featured_image_url`   | string      | The Mobilize-hosted image URL for the event. Must be generated using the [Upload images](#upload-images) endpoint. | No
| `tag_ids`              | int[]       | Array of tag ids to be applied to the event. Any existing tags that are not present in the `tag_ids` array will be deleted.                                   | No
| `instructions`         | string      | Private instructions sent to attendees of the event after signing up. If omitted, will leave the existing value. | No

### Request body example

    {
        "title": "Example",
        "description": "example",
        "timezone": "America/New_York",
        "timeslots": [
            {
                "id": 2000,
                "start_date": 1576774900,
                "end_date": 1576782100,
                "instructions": "Updated instructions for timeslot with id 2000"
            },
            {
                "start_date": 1576861300,
                "end_date": 1576868500,
                "instructions": "New instructions for new timeslot"
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
        "visibility": "PUBLIC",
        "contact": {
            "email_address": "replyto@thisemail.com"
        },
        "tag_ids": [2, 34],
        "instructions": "Definitely bring a hat"
    }


### Response
If the `event_id` does not identify an existing event, the endpoint will return a `404 Not Found` response.

`data` will contain the updated Event object on a successful request.

## Delete event

Status: RESTRICTED

Please email support@mobilize.us to request access to this endpoint.

Delete an in-person event for an organization.

Requires authentication: Yes

### Request
`DELETE /v1/organizations/:organization_id/events/:event_id`

### Request params
* `send_update_notifications`: Defaults to `true`. Set this to `false` to skip notifications. Defaults to sending attendees notifications when the event is deleted.
  *  `DELETE /v1/organizations/:organization_id/events/:event_id?send_update_notifications=false`

### Response
On success, the endpoint will return with status code `200 OK`.

# People
## Person object
| Field              | Type      | Description                           |
| ------------------ | --------- | ------------------------------------- |
| `id`               | int       | Equivalent to `person_id` below       |
| `person_id`        | int       | Due to internal database reasons, this will eventually be deprecated and be `null` for new Persons |
| `user_id`          | int       | This will be the canonical id going forward |
| `created_date`     | int       | Unix timestamp                        |
| `modified_date`    | int       | Unix timestamp                        |
| `given_name`       | string    | First name                            |
| `family_name`      | string    | Last name                             |
| `email_addresses`  | Email[]   | Array of length 1 with an email. (NB: See Email details below for exceptions.) |
| `phone_numbers`    | Phone[]   | Array of length 1 with a phone number |
| `postal_addresses` | Address[] | Array of length 1 with a zip code     |
| `sms_opt_in_status`| enum      | For the current organization; one of `UNSPECIFIED`, `OPT_IN` or `OPT_OUT` |
| `blocked_date`     | int       | Unix timestamp. Will be `null` if the person is not blocked by the viewing organization |

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
`GET /v1/organizations/:organization_id/people`

### Request params

- `updated_since`: Unix timestamp to filter by Persons’ `modified_date`

### Response
`data` is an array of Person objects.

# Attendances
## Attendance object

| Field                             | Type                       | Description                                                                 | Omission notes                                                                                                              |
| --------------------------------- | -------------------------- | --------------------------------------------------------------------------- |-----------------------------------------------------------------------------------------------------------------------------|
| `id`                              | int                        |                                                                             | If the requesting organization is independent and the event’s organization is coordinated, this is omitted.                 |
| `created_date`                    | int                        | Unix timestamp                                                              |                                                                                                                             |
| `modified_date`                   | int                        | Unix timestamp                                                              |                                                                                                                             |
| `person`                          | Person                     | Person who attended the event                                               |                                                                                                                             |
| `event`                           | Event                      | Associated event                                                            | If the requesting organization is independent and the event’s organization is coordinated, all but `event_type` is omitted. |
| `timeslot`                        | Timeslot                   | Selected timeslot on event                                                  | If the requesting organization is independent and the event’s organization is coordinated, `id` is omitted.                 |
| `sponsor`                         | Organization               | The promoting organization if it exists, otherwise the event’s organization | If the requesting organization is coordinated and the promoting organization is independent, this is omitted.               |
| `status`                          | enum                       | `REGISTERED`, `CANCELLED`, or `CONFIRMED`                                   |                                                                                                                             |
| `attended`                        | bool                       | Whether the person actually attended or not. Will be `null` if not set.     |                                                                                                                             |
| `referrer`                        | Referrer                   | UTM tracking information                                                    | If this attendance came through a promotion and the requesting organization is not the promoter, this is omitted.           |
| `custom_signup_field_values`      | CustomSignupFieldValue[]   | Any additional fields collected on event signups                            | If the requesting organization is not the event owner or co-owner, this is omitted.                                         |

### Referrer

| Field          | Type   | Description |
| -------------- | ------ | ----------- |
| `utm_source`   | string |             |
| `utm_medium`   | string |             |
| `utm_campaign` | string |             |
| `utm_term`     | string |             |
| `utm_content`  | string |             |
| `url`          | string |             |

### CustomSignupFieldValue

| Field               | Type   | Description                                                                                                 |
| ------------------- | ------ | ----------------------------------------------------------------------------------------------------------- |
| `custom_field_id`   | int    | The id of the custom signup field to which the attendee responded                                           |
| `custom_field_name` | string | The unique name used internally for the custom field                                                        |
| `text_value`        | string | Text value of the attendee's response. Exactly one of `text_value` and `boolean_value` will be non-null     |
| `boolean_value`     | bool   | Boolean value of the attendee's response. Exactly one of `text_value` and `boolean_value` will be non-null  |

## Get an organization attendance

Status: LIVE

Fetch a single attendance for an organization.

Requires authentication: Yes

### Request
`GET /v1/organizations/:organization_id/attendances/:attendance_id`

### Response
`data` is the single Attendance object.

## List organization attendances

Status: LIVE

Fetch all attendances which were either promoted by the organization or were for events owned by the organization

Requires authentication: Yes

### Request
`GET /v1/organizations/:organization_id/attendances`

### Request params

- `updated_since`: Unix timestamp to filter by Attendances’ `modified_date`

### Response
`data` is an array of Attendance objects.

## List organization attendances for an event

Status: LIVE

Featch all attendances for the given event if the event is either promoted by the organization or owned by the organization

Requires authentication: Yes

### Request
`GET /v1/organizations/:organization_id/events/:event_id/attendances`

### Response
`data` is an array of Attendance objects.

## Create organization event attendance

Status: RESTRICTED

Please email support@mobilize.us to request access to this endpoint.

This endpoint creates a new signup for a given person and event timeslot. If multiple timeslots are provided, an Attendance object will be created for each timeslot. The person is matched and deduplicated by their email address.


Requires authentication: Yes

### Request
`POST /v1/organizations/:organization_id/events/:event_id/attendances`

### Request body

| Field             | Type     | Description                                                                                                                                                                                                                                                                                                                                     | Required                       |
| ----------------- | -------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------ |
| person            | AttendancePerson   | Person attendance object. A person is matched and deduplicated by their email address; a record for the person will be created if the `email_address` is not already associated with a person, otherwise the remaining person fields will be updated. | Yes    |
| sms_opt_in        | string   | One of `OPT_IN`, `UNSPECIFIED`.                                                                                                                                                                                                                                                                                                                 | No (defaults to `UNSPECIFIED`) |
| transactional_sms_opt_in | string | One of `OPT_IN`, `UNSPECIFIED`. `UNSPECIFIED` means the person will not be opted in to transactional text messages, but if the person has opted in previously, will not remove that opt in. | No (defaults to `OPT_IN`) |
| timeslots         | Timeslot[]    | Array of `timeslot` objects. Must be existing timeslots for the given Event.                                                                                                                                                                                                                                                               | Yes                            |
| referrer          | Referrer | Referrer object used to add additional tracking to request.                                                           | No                             |

### Person attendance object

| Field            | Type   | Description | Required
| ---------------- | ------ | ----------- | --------
| `given_name`     | string |             | Yes
| `family_name`  | string |             | Yes
| `email_address`  | string |             | Yes
| `phone_number`   | string |             | Yes, but you can pass the empty string if you don't have the phone number
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

### Request body example

    {
        "person": {
            "given_name": "myfirstname",
            "family_name": "mylastname",
            "email_address": "email@email.com",
            "phone_number": "1234567890",
            "postal_code": "11111"
        },
        "sms_opt_in": "OPT_IN",
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

### Response body example

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


# Affiliation


## Create organization affiliations

Status: RESTRICTED

Please email support@mobilize.us to request access to this endpoint.

This endpoint creates a new affiliation between the given person and organization, or updates a person’s contact information if an affiliation already exists. The person is matched and deduplicated by their email address.

Requires authentication: Yes

### Request
`POST /v1/organizations/:organization_id/affiliations`

### Request body
| Field                           | Type   | Description                                       | Required |
| ------------------------------- | ------ | ------------------------------------------------- | -------- |
| given_name                      | string | The first name of the person.                     | Yes      |
| family_name                     | string | The last name of the person.                      | Yes      |
| email_address                   | string | The email address of the person.                  | Yes      |
| phone_number                    | string | The phone number of the person.                   | Yes      |
| address_line_1                  | string | The first address line of the person’s location.  | No       |
| address_line_2                  | string | The second address line of the person’s location. | No       |
| locality                        | string | The city of the person’s location.                | No       |
| region                          | string | The U.S. state of the person’s location.          | No       |
| postal_code                     | string | The zipcode of the person’s location.             | Yes      |
| sms_opt_in                      | string | One of `OPT_IN`, `UNSPECIFIED`.                   | Yes      |
| transactional_sms_opt_in        | string | One of `OPT_IN`, `UNSPECIFIED`. `UNSPECIFIED` means the person will not be opted in to transactional text messages, but if the person has opted in previously, will not remove that opt in. | No (defaults to `OPT_IN`) |


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
        "sms_opt_in": "OPT_IN",
        "transactional_sms_opt_in": "UNSPECIFIED"
    }

### Response

If any required fields are missing or contain invalid values, the endpoint will return a 400 Bad Request with references to the invalid fields.

On a successful request, the endpoint will return a 201 Created status code if the person record was created, a 200 No Content result if the person record was updated, and the affected Affiliation object.

# Images

## Upload images

Status: LIVE

This endpoint uploads an image to Mobilize and returns the Mobilize-hosted image URL. This URL can then be used as the `featured_image_url` when creating or updating an Event.

This endpoint accepts the `multipart/form-data` content type.

Requires authentication: Yes

### Request
`POST /v1/images`

### Request body
| Field             | Type   | Description                                       | Required |
| ----------------- | ------ | ------------------------------------------------- | -------- |
| file              | File   | The image to upload.                              | Yes      |
| file_name         | string | An optional name for the file; if not provided, will be inferred from the file's name.  | No      |

### Response
`data` contains the Mobilize-hosted image URL, which can then be used as the `featured_image_url` when creating or updating events.

# Enums

## Get enums

Status: LIVE

Fetch a list of enums and their available values to be passed to event and attendance creation endpoints.

Requires authentication: No

### Request
`GET /v1/enums`

### Response
`data` contains keys and values for the `event_type`, `visibility`, `accessibility_status`, `approval_status`, `timezone`, `sms_opt_in`, and `transactional_sms_opt_in` fields.

# Staging and Development

For testing and development purposes, there is also a staging api server:

 * Web Interface: https://staging.mobilize.us
 * Api entrypoint: `staging-api.mobilize.us/v1`

# Changelog
**2024-03-27**
- Allow filtering by `state` in [List organization events](#list-organization-events)

**2023-05-11**
- Omit referrer fields (UTM fields) from Attendances when the Attendance came through promotion and the requesting organization is not the promoter.

**2021-09-02**
- Update to use [cursor based pagination](#cursor) everywhere

**2021-06-01**
- Add [cursor based pagination](#cursor)

**2021-04-01**
- Add `custom_signup_field_values` to [Attendance](#attendance-object) object

**2021-01-11**
- Add [Get enums](#get-enums) endpoint

**2020-10-16**
- Remove long-deprecated `summary` field from Event object and related create/update requests. It was not required, and has not appeared anywhere in the product for some time.

**2020-10-15**
- Clarify that phone number is required but can be the empty string in [Create organization event attendance](#create-organization-event-attendance)
- Add new `event_type` options [Event object](#event-object): `WORKSHOP`, `PETITION`, `AUTOMATED_PHONE_BANK`, `LETTER_WRITING`, `LITERATURE_DROP_OFF`, `VISIBILITY_EVENT`

**2020-09-04**
- Update [List organization events](#list-organization-events) to return virtual and in-person events when a `zipcode` is provided. (Previously, including `zipcode` would exclude virtual events.) In order to limit a search to only in-person events, set `is_virtual=false`.

**2020-09-03**
- Fix typos in [Create organization affiliations](#create-organization-affiliations) and [Create organization event attendance](#create-organization-event-attendance) that references `sms_opt_in_status` and `transactional_sms_opt_in_status`. The correct field name are `sms_opt_in` and `transactional_sms_opt_in`.

**2020-08-14**
- Update [Create organization event attendance](#create-organization-event-attendance) to allow adding attendances to past timeslots

**2020-07-22**
- Add `is_virtual` to [Create event](#create-event) and [Update event](#update-event)
- Make `location` optional for virtual events for [Create event](#create-event) and [Update event](#update-event)

**2020-06-26**
- Add `approval_status` to [Event object](#event-object)
- Allow filtering by `approval_status` in [List organization events](#list-organization-events)

**2020-06-24**
- Allow filtering by `event_campaign_id` in [List organization events](#list-organization-events)

**2020-06-15**
- Add `instructions` to [Event object](#event-object) and [Timeslot object](#timeslot), and update [event creation](#create-event) and [event update](#update-event) to support `instructions` as well
- Add `instructions` and `event_campaign` to sample requests

**2020-03-31**
- Add `is_virtual` to [Event object](#event-object)
- Virtual events can now have locations; Event endpoints will now include locations for virtual events if they are available.

**2020-03-10**
- Notice of upcoming changes to the API. See [details here](https://github.com/mobilizeamerica/api/pull/64).
  - Deprecate [List All Public Events](#list-all-public-events) and [List Deleted Public Events](#list-deleted-public-events)
  - Remove unused `max_dist` and `zipcode` filters from our List Deleted Organization Events
  - Add explainer on `results_limited_to` and fields that trigger it

**2020-03-11**
- Include `blocked_date` in [Person objects](#person-object). Available on [list organization people](#list-organization-people) and attendances endpoints.

**2020-02-25**
- Update [event creation](#create-event) and [event update](#update-event) to no longer delete past timeslots when they're not passed to these requests, and clarify documentation on how to pass in timeslots

**2020-02-20**
- Update [Create organization affiliations](#create-organization-affiliations) to allow specifying transactional sms opt in status to `UNSPECIFIED`.
- Update [Create organization event attendance](#create-organization-event-attendance) to allow specifying transactional sms opt in status to `UNSPECIFIED`.

**2020-02-13**
- Update [create event](#create-event) and related resources to include information about `max_attendees`.

**2020-01-27**
- Update [list events](#list-events) and [list organization events](#list-organization-events) to
  specify that search results with `zipcode` param maxes out at 1000 results regardless of pagination.

**2020-01-16**
- Update [list organization events](#list-organization-events) to return a 403 error if the the user
  queries for `visibility=PRIVATE` without permissions for the org or the request is unauthenticated.

**2020-01-03**
- Clarify [list organization events](#list-organization-events) and [list deleted organization events](#list-deleted-organization-events) endpoint docs re: event visibility behavior.

**2019-12-11**
- Update [event update](#update-event) to include an optional list of `tag_ids`

**2019-12-09**
- Update [Person attendance object](#person-attendance-object) so that phone number is not required anymore.
- Update [event creation](#create-event) to include an optional list of `tag_ids`

**2019-12-05**
- Add [single attendance](#get-an-organization-attendance) and [attendances by event](#list-organization-attendances-for-an-event) endpoints
- Add endpoint to upload an image and retrieve its Mobilize-hosted URL
- Allow setting an image URL when creating an updating an event

**2019-11-04**
- Add [single event](#get-an-event) and [single organization event]($get-an-organization-event) endpoints

**2019-10-31**
- Add new `event_type` options [Event object](#event-object): `COMMUNITY_CANVASS`, `SIGNATURE_GATHERING`, `CARPOOL`,

**2019-09-25**
- Add `event_campaign` on the [Event object](#event-object), and the associated [EventCampaign object](#eventcampaign).

**2019-08-30**
- Add new `event_type` options to [Event object](#event-object): `RALLY`, `TOWN_HALL`, `OFFICE_OPENING`, `BARNSTORM`, `SOLIDARITY_EVENT`

**2019-08-19**
- Allow [public events](#list-all-public-events) and [organization events](#list-organization-events) to be filtered by `tag_id`

**2019-08-15**
- Introduce [Tag object](#tag)
- Add `tags` to [Event object](#event-object)

**2019-07-23**
- Document `ADVOCACY_CALL` event type, and clarify that the write APIs do not currently accept it

**2019-07-22**
- Change `accessibility_status` to not required for Event POST/PUT requests.
- Expand list of event types to include `DEBATE_WATCH_PARTY`
- Document `contact` on request body of [create event](#create-event) and [update event](#update-event) endpoints

**2019-07-19**
- Add `accessibility_status` and `accessibility_notes` to [Event object](#event-object)

**2019-07-18**
- Add `country` to [Location object](#location)

**2019-06-25**
- Add `owner_user_id` to Contact object
- Update doc to clarify that `contact` is a field on Event object (this was previously the behavior, but it was missing from the documentation)

**2019-06-13**
- Add `person_id` and `user_id` to [Person object](#person-object)

**2019-06-12**
- Add `is_virtual` and `event_types` filter params to [list public events](#list-all-public-events) and [list organization events](#list-organization-events) endpoints.

**2019-06-04**
- Add `org_type` to Organization object
- Add `virtual_action_url` and `address_visibility` to Event type
- Change `address_lines` and `venue` for Location objects where the location is private

**2019-05-14**
- Add `is_full` to Timeslot
- Add `exclude_full` filter option on public events and organization events endpoints

**2019-04-03**
- Remove (unused) API endpoint GET `/v1/organizations/:organization_id/people/:person_id/attendances`

**2019-03-12**
- Update API entrypoint URL from `https://www.mobilize.us/api/v1` to
  `https://api.mobilize.us/v1`. The old version will continue to work (as will
  `https://events.mobilizeamerica.io/api/v1`) until Sept. 1, 2019.
- Update references to support email (`support@mobilizeamerica.io` to `support@mobilize.us`).

**2019-03-06**
- Update references to website url (`https://events.mobilizeamerica.io` to `https://www.mobilize.us`)

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
