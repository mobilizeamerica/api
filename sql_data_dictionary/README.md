# MobilizeAmerica SQL Mirror - Data Dictionary

The MobilizeAmerica SQL mirror is a feature we offer to our customers that enables them to access their
raw data. Our application has several CRM integrations and the ability to perform CSV exports, but
customers with sophisticated data operations may find it more convenient to extract data from our SQL
mirror. Use cases include performing analytics, creating dashboards and reports, and loading data into
data warehouses or Business Intelligence tools.

The data is continuously updated from our application's database in real-time so your queries will
always return the most current data, just like when you query the API.

Below you will find a comprehensive Data Dictionary of all the SQL views we currently provide with a
description of each field.

---

# Table of Contents
- [MobilizeAmerica SQL Data Dictionary](#mobilizeamerica-sql-data-dictionary)
- [Table of Contents](#table-of-contents)
- [SQL Views](#sql-views)
  - [Events](#events)
  - [Participations](#participations)
  - [Timeslots](#timeslots)
- [Changelog](#changelog)

# SQL Views

## Events

The `events` view includes events owned by the Organization and events owned by Organizations that are being promoted. Some fields will be masked for promoted events.

| column name | type | description |
| ----------- | ---- | ----------- |
| id | integer | The event's primary key |
| title | varchar(100) | The public name of the event |
| summary | varchar(100) | The public subheading of the event |
| description | text | Long-form HTML description of the event |
| featured_image_url | varchar(1000) | Path to the image for the event |
| high_priority | boolean | Whether the event is marked high priority for the provided organization |
| organization_id | integer | Unique identifier of the Organization that owns the event |
| organization__name | varchar(100) | The public-facing name of the organization |
| organization__slug | citext | The URL-safe string for this organization |
| organization__is_coordinated | boolean | Whether this is a coordinated-side organization for campaign finance purposes. Opposite of `is_independent` |
| organization__is_independent | boolean | Whether this is an independent-side organization for campaign finance purposes. Opposite of `is_coordinated` |
| organization__race_type | integer | One of `GOVERNOR`, `CONGRESSIONAL`, `SENATE`, `STATE_SENATE`, `STATE_LEG`, `SEC_STATE`, `ATTY_GENERAL`, `OTHER_LOCAL`, `OTHER_STATEWIDE`. New options may be added. `null` if not a campaign. |
| organization__is_primary_campaign | boolean | If the campaign is a primary. `false` if not a campaign. |
| organization__state | varchar(2) | The two-letter code of the state the organization is in. May be empty. |
| organization__district | varchar(100) | The district the organization is in, e.g., `14`. May refer to different districts depending on the value of race_type. |
| organization__candidate_name | varchar(100) | The name of the candidate this organization represents. May be empty. |
| organization__modified_date | timestamptz | Timestamp that the organization was last modified |
| location__is_private | boolean | Whether or not the location fields are visible to the public |
| location__venue | varchar(250) | The name of the location, e.g., "Campaign HQ" or "Starbucks" |
| location__address_line_1 | varchar(250) | The first line of the address |
| location__address_line_2 | varchar(250) | The second line of the address |
| location__locality | varchar(250) | The city |
| location__region | varchar(2) | The two-character state code |
| location__postal_code | varchar(10) | The zip code |
| location__lat | numeric(10, 7) | The latitude of the location or null if geocoding failed or has not yet completed |
| location__lon | numeric(10, 7) | The longitude of the location or null if geocoding failed or has not yet completed |
| location__modified_date | timestamptz | Timestamp that the location was last updated |
| location__congressional_district | varchar(50) | The congressional district that the location is in or null if geocoding failed or has not yet completed (a full street address is required to geocode districts) |
| location__state_leg_district | varchar(50) | The state lower house district that the location is in or null if geocoding failed or has not yet completed (a full street address is required to geocode districts)|
| location__state_senate_district | varchar(50) | The state upper house district that the location is in or null if geocoding failed or has not yet completed (a full street address is required to geocode districts)|
| timezone | varchar(100) |	A timezone database string for the event, e.g., `America/New_York` |
| event_type | varchar | The type of event. One of: `ADVOCACY_CALL`, `CANVASS`, `COMMUNITY`, `FRIEND_TO_FRIEND_OUTREACH`, `FUNDRAISER`, `HOUSE_PARTY`, `MEET_GREET`, `MEETING`, `OTHER`, `PHONE_BANK`, `TEXT_BANK`, `TRAINING`, `VOTER_REG` |
| browser_url | text | Canonical URL of the event |
| created_date | timestamptz | Timestamp that the event was created |
| modified_date | timestamptz | Timestamp that the event was last updated |
| deleted_date | timestamptz | Timestamp that the event was deleted (null if not deleted) |
| visibility | varchar | The visibility of the event, one of: `PUBLIC`, `PRIVATE` |
| created_by_volunteer_host | boolean | Whether the event was created by a volunteer host using our distributed organizing tool or not |
| contact__name | varchar(100) | The name of the contact for the event |
| contact__email_address | varchar(254) | The email address of the contact for the event (reply-to email) |
| contact__phone_number | varchar(100) | The phone number of the contact for the event |
| approval_status | varchar | Status of an event if it requires admin approval. One of: `APPROVED`, `NEEDS_APPROVAL`, `REJECTED`, `NEEDS_HOST_VERIFICATION` |
| rejection_message | text | The reason that an admin provided for rejecting an event |
| referrer__utm_source | varchar | Value of the `utm_source` parameter for the url the host used to create the distributed organizing event. `null` for promoted events and non-distributed organizing events. |
| referrer__utm_medium | varchar | Value of the `utm_medium` parameter for the url the host used to create the distributed organizing event. `null` for promoted events and non-distributed organizing events.   |
| referrer__utm_campaign | varchar | Value of the `utm_campaign` parameter for the url the host used to create the distributed organizing event. `null` for promoted events and non-distributed organizing events. |
| referrer__utm_term | varchar | Value of the `utm_term` parameter for the url the host used to create the distributed organizing event. `null` for promoted events and non-distributed organizing events. |
| referrer__utm_content | varchar | Value of the `utm_content` parameter for the url the host used to create the distributed organizing event. `null` for promoted events and non-distributed organizing events. |
| referrer__url | varchar | Value of the url the host used to create the distributed organizing event. `null` for promoted events and non-distributed organizing events. |
| owner_id | integer | Unique identifier of the User who currently owns the event. The following `owner__` fields are `null` for promoted events. |
| owner__modified_date | timestamptz | Timestamp the User was updated |
| owner__given_name | varchar(255) | User's first name |
| owner__family_name | varchar(255) | User's last name |
| owner__email_address | citext | User's email address |
| owner__phone_number | varchar(15) | User's phone number |
| owner__postal_code | varchar(10) | User's zip code |
| creator_id | integer | Unique identifier of the User who created the event. The following `creator__` fields are `null` for promoted events. |
| creator__modified_date | timestamptz | Timestamp the User was updated |
| creator__given_name | varchar(255) | User's first name |
| creator__family_name | varchar(255) | User's last name |
| creator__email_address | citext | User's email address |
| creator__phone_number | varchar(15) | User's phone number |
| creator__postal_code | varchar(10) | User's zip code |
| reviewed_date | timestamptz | The timestamp that the admin reviewed the event, updating its `approval_status` |
| reviewed_by_id | integer | Unique identifier of the User who reviewed the event. The following `reviewed_by__` fields are `null` for promoted events. |
| reviewed_by__modified_date | timestamptz | Timestamp the User was updated |
| reviewed_by__given_name | varchar(255) | User's first name |
| reviewed_by__family_name | varchar(255) | User's last name |
| reviewed_by__email_address | citext | User's email address |
| reviewed_by__phone_number | varchar(15) | User's phone number |
| reviewed_by__postal_code | varchar(10) | User's zip code |

## Participations

The `participations` view contains information about a volunteer's individual signup to a specific
timeslot for an event. This data is referred to as `Attendances` in the Public API. Participations
that originated from an event signup page in another organization's feed are also displayed (when the
viewing organization is the org in the `affiliation_id`), but some of the fields will be masked.

| column name | type | description |
| ----------- | ---- | ----------- |
| id | integer | The participation's primary key |
| created_date | timestamptz | Timestamp that the participation was created |
| modified_date | timestamptz | Timestamp that the participation was last updated |
| user_id | integer | Unique identifier of the User who signed up for this participation |
| user__modified_date | timestamptz | Timestamp the User was last updated |
| user__given_name | varchar(255) | User's current first name |
| user__family_name | varchar(255) | User's current last name |
| user__email_address | citext | User's current email address |
| user__phone_number | varchar(15) | User's current phone number |
| user__postal_code | varchar(10) | User's current zip code |
| event_id | integer | Foreign Key to the related Event. This field is `null` if the organization and the affiliated organization are cross firewall. |
| timeslot_id | integer | Foreign Key to the related Timeslot. This field is `null` if the organization and the affiliated organization are cross firewall. |
| override_start_date | timestamptz | Time that an event's start was overridden to. This only applies for "pick a time" virtual events that were synced from VAN. |
| override_end_date | timestamptz | Time that an event's end was overridden to. This only applies for "pick a time" virtual events that were synced from VAN. |
| organization_id | integer | Unique identifier of the Organization that owns the event. This may be the same organization that is querying the view or an organization that it is promoting. This field is `null` if cross firewall unless the viewing organization owns the event. |
| organization__name | varchar(100) | The public-facing name of the organization. This field is `null` if cross firewall unless the viewing organization owns the event. |
| organization__slug | citext | The URL-safe string for the organization. This field is `null` if cross firewall unless the viewing organization owns the event. |
| affiliation_id | integer | Unique identifier of the Organization whose feed the User signed up for the event on. This may be the same as the `organization_id` that owns the event. This field is `null` if cross firewall unless the viewing organization is the affiliated org. |
| affiliation__name | varchar(100) | The public-facing name of the organization. This field is `null` if cross firewall unless the viewing organization is the affiliated org. |
| affiliation__slug | citext | The URL-safe string for the organization. This field is `null` if cross firewall unless the viewing organization is the affiliated org. |
| participation_status | varchar | The user's RSVP status before the event has occurred. One of: `REGISTERED`, `CANCELLED`, `CONFIRMED` |
| attended | boolean | Whether the volunteer actually attended or not. Will be `null` if not set. |
| experience_feedback_type | varchar | The user-reported feedback on the event. One of: `APPROVED_OF_SHIFT`, `DISAPPROVED_OF_SHIFT`, `DID_NOT_ATTEND` |
| experience_feedback_text | text | The user-reported qualitative feedback on the event. This field is `null` if the organization that owns the event and the affiliated organization are cross firewall. |
| referrer__utm_source | varchar | Value of the `utm_source` parameter in the url that was used to sign up for the event. `null` for promoted events. |
| referrer__utm_medium | varchar | Value of the `utm_medium` parameter in the url that was used to sign up for the event. `null` for promoted events. |
| referrer__utm_campaign | varchar | Value of the `utm_campaign` parameter in the url that was used to sign up for the event. `null` for promoted events. |
| referrer__utm_term | varchar | Value of the `utm_term` parameter in the url that was used to sign up for the event. `null` for promoted events. |
| referrer__utm_content | varchar | Value of the `utm_content` parameter in the url that was used to sign up for the event. `null` for promoted events. |
| referrer__url | varchar | Value of the url used to sign up for the event. `null` for promoted events. |
| email_at_signup | citext | The email address the user entered when signing up |
| given_name_at_signup | varchar(100) | The first name the user entered when signing up |
| family_name_at_signup | varchar(100) | The last name the user entered when signing up |
| phone_number_at_signup | varchar(20) | The phone number the user entered when signing up |
| postal_code_at_signup | varchar(10) | The zip code the user entered when signing up |

## Timeslots

The `timeslots` view contains information about the start and end times for event timeslots (shifts).
Only timeslots for events owned by the organization or promoted by the organization are included.

| column name | type | description |
| ----------- | ---- | ----------- |
| id | integer | The timeslot's primary key |
| start_date | timestamptz | Time that the timeslot begins |
| end_date | timestamptz | Time that the timeslot ends |
| event_id | integer | Foreign Key to the Event that the timeslot belongs to  |
| created_date | timestamptz | Time that the timeslot was created |
| modified_date | timestamptz | Time that the timeslot was last updated |
| deleted_date | timestamptz | Time that the timeslot was deleted. `null` if not deleted. |
| max_attendees | integer | Maximum attendees for the timeslot, if set. `null` if the Timeslot is from a promoted event. |


# Changelog

**2019-05-14**
- Add `max_attendees` to `timeslots` view
