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
- [MobilizeAmerica SQL Mirror - Data Dictionary](#mobilizeamerica-sql-mirror---data-dictionary)
- [Table of Contents](#table-of-contents)
- [SQL Views](#sql-views)
  - [Events](#events)
  - [Participations](#participations)
  - [Timeslots](#timeslots)
  - [Sms Opt Ins](#sms-opt-ins)
  - [Event Tags](#event-tags)
  - [Users](#users)
  - [Affiliations](#affiliations)
  - [VAN Views](#van-views)
  - [VAN Events](#van-events)
  - [VAN Shifts](#van-shifts)
  - [VAN Signups](#van-signups)
  - [VAN Persons](#van-persons)
  - [VAN Locations](#van-locations)
  - [Event Co-Hosts](#event-co-hosts)
  - [Organizations](#organizations)
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
| location__region | varchar(2) | The two-character state code (one of the 50 U.S. states or `DC` for Washington, D.C.) |
| location__country | varchar(2) | The [ISO-3166-1 alpha-2](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2) country code; note that U.S. territories and commonwealths have distinct country codes, e.g., Puerto Rico is `PR` |
| location__postal_code | varchar(10) | The zip code |
| location__lat | numeric(10, 7) | The latitude of the location or null if geocoding failed or has not yet completed |
| location__lon | numeric(10, 7) | The longitude of the location or null if geocoding failed or has not yet completed |
| location__modified_date | timestamptz | Timestamp that the location was last updated |
| location__congressional_district | varchar(50) | The congressional district that the location is in or null if geocoding failed or has not yet completed (a full street address is required to geocode districts) |
| location__state_leg_district | varchar(50) | The state lower house district that the location is in or null if geocoding failed or has not yet completed (a full street address is required to geocode districts)|
| location__state_senate_district | varchar(50) | The state upper house district that the location is in or null if geocoding failed or has not yet completed (a full street address is required to geocode districts)|
| timezone | varchar(100) |	A timezone database string for the event, e.g., `America/New_York` |
| event_type | varchar | The type of event. One of: `ADVOCACY_CALL`, `CANVASS`, `COMMUNITY`, `DEBATE_WATCH_PARTY`, `FRIEND_TO_FRIEND_OUTREACH`, `FUNDRAISER`, `HOUSE_PARTY`, `MEET_GREET`, `MEETING`, `OTHER`, `PHONE_BANK`, `TEXT_BANK`, `TRAINING`, `VOTER_REG` |
| browser_url | text | Canonical URL of the event |
| created_date | timestamptz | Timestamp that the event was created |
| modified_date | timestamptz | Timestamp that the event was last updated |
| deleted_date | timestamptz | Timestamp that the event was deleted (null if not deleted) |
| visibility | varchar | The visibility of the event, one of: `PUBLIC`, `PRIVATE` |
| created_by_volunteer_host | boolean | Whether the event was created by a volunteer host using our distributed organizing tool or not |
| registration_mode | varchar | Whether an event allows or requires donations on signup. `DEFAULT` means no donations allowed. `DONATE_TO_RSVP` means donations required. `DONATE_TO_RSVP_OPTIONAL` means donations allowed but not required. |
| van_name | varchar(500) | A custom name of the event in VAN, if set. If not set, the name of the event in VAN defaults to the `title`. Always `null` for promoted events. |
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
| accessibility_notes | varchar | Additional details about accessibility status and accommodations at the venue |
| accessibility_status | varchar | The degree of compliance with the Americans with Disabilities Act. One of: `ACCESSIBLE`, `NOT_ACCESSIBLE`, `NOT_SURE`, or `UNKNOWN` |
| is_virtual | boolean | If `true`, the event is a Virtual event. If `false`, it is an In-Person event. |
| event_campaign_id | integer | Unique identifier of the Event Campaign that the event belongs to |
| event_campaign__slug | citext | The URL-safe string for the Event Campaign. This slug is unique for the organization. |
| virtual_action_url | varchar | The URL the event redirects to if it's an unshifted virtual event. |

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
| user__blocked_date | timestamptz | Timestamp that the User was blocked by the organization. `null` if the User is not blocked |
| event_id | integer | Foreign Key to the related Event. Unless the querying org is the event owner or co-owner, this field is `null` if the organization and the affiliated organization are cross firewall. |
| timeslot_id | integer | Foreign Key to the related Timeslot. Unless the querying org is the event owner or co-owner, this field is `null` if the organization and the affiliated organization are cross firewall. |
| override_start_date | timestamptz | Time that an event's start was overridden to. This only applies for "pick a time" virtual events that were synced from VAN. |
| override_end_date | timestamptz | Time that an event's end was overridden to. This only applies for "pick a time" virtual events that were synced from VAN. |
| organization_id | integer | Unique identifier of the Organization that owns the event. This may be the same organization that is querying the view or an organization that it is promoting. This field is `null` if cross firewall unless the viewing organization owns the event. |
| organization__name | varchar(100) | The public-facing name of the organization. This field is `null` if cross firewall unless the viewing organization owns the event. |
| organization__slug | citext | The URL-safe string for the organization. This field is `null` if cross firewall unless the viewing organization owns the event. |
| affiliation_id | integer | Unique identifier of the Organization whose feed the User signed up for the event on. This may be the same as the `organization_id` that owns the event. This field is `null` if cross firewall unless the viewing organization is the affiliated org. |
| affiliation__name | varchar(100) | The public-facing name of the organization. This field is `null` if cross firewall unless the viewing organization is the affiliated org. |
| affiliation__slug | citext | The URL-safe string for the organization. This field is `null` if cross firewall unless the viewing organization is the affiliated org. |
| status | varchar | The user's RSVP status before the event has occurred. One of: `REGISTERED`, `CANCELLED`, `CONFIRMED`. Unless the querying org is the event owner or co-owner, this field is `null` if the organization that owns the event and the affiliated organization are cross firewall. |
| attended | boolean | Whether the volunteer actually attended or not. Will be `null` if not set. Unless the querying org is the event owner or co-owner, this field is `null` if the organization that owns the event and the affiliated organization are cross firewall. |
| experience_feedback_type | varchar | The user-reported feedback on the event. One of: `APPROVED_OF_SHIFT`, `DISAPPROVED_OF_SHIFT`, `DID_NOT_ATTEND`. Unless the querying org is the event owner or co-owner, this field is `null` if the organization that owns the event and the affiliated organization are cross firewall. |
| experience_feedback_text | text | The user-reported qualitative feedback on the event. Unless the querying org is the event owner or co-owner, this field is `null` if the organization that owns the event and the affiliated organization are cross firewall. |
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
| custom_field_values | json | The custom field values collected on the signup, if present, otherwise null. Contains a list of objects, each containing the custom field ID, name, and boolean or text value collected for the participation. |
| event_type_name | varchar | The string value that corresponds with the `event_type` integer. |

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

## Sms Opt Ins

The `sms_opt_ins` view contains information about which volunteers have opted in to sms communication with the given organization.

| column name | type | description |
| ----------- | ---- | ----------- |
| id | integer | The primary key |
| created_date | timestamptz | Time that the opt in information was first collected |
| modified_date | timestamptz | Time that the opt in information was last updated |
| sms_opt_in_status | varchar | The user's opt in status. One of: `UNSPECIFIED`, `OPT_IN`, `OPT_OUT` |
| user_id | integer | Unique identifier of the User whose opt in information this represents |
| user__phone_number | varchar(20) | User's current phone number. Note: this may not match the phone we display on participations as `user__phone_number` but this is the phone the opt in was collected on |
| organization_id | integer | Unique identifier of the Organization whose opt in this is for |

## Event Tags

The `event_tags` view contains information about what tags have been applied to events. Only tags for events owned or promoted by the organization are included.

| column name | type | description |
| ----------- | ---- | ----------- |
| id | integer | The primary key of the event-tag relation |
| created_date | timestamptz | Time that the tag was added to the event |
| modified_date | timestamptz | Time that the event-tag relation was last updated |
| deleted_date | timestamptz | Time that the tag was removed from the event. `null` if not removed. |
| event_id | integer | Foreign Key to the Event this tag relation belongs to |
| tag_id | integer | Unique identifier of the tag |
| tag__name | citext | Name of the tag |

## Users

The `users` view contains information about members of the Organization. Note that for schemas used to monitor multiple organizations, `membership_id` is the only guaranteed unique identifier for this table.

| column name | type | description |
| ----------- | ---- | ----------- |
| id | integer | The primary key of the User |
| created_date | timestamptz | Time that the User was created |
| modified_date | timestamptz | Time that the User was last updated |
| given_name | varchar(255) | User's first name |
| family_name | varchar(255) | User's last name |
| email_address | citext | User's email address |
| phone_number | varchar(15) | User's phone number |
| postal_code | varchar(10) | User's zip code |
| blocked_date | timestamptz | Time that the User was blocked by the organization. `null` if the User is not blocked |
| membership_id | integer | Foreign Key to the Membership of this User in the Organization |
| membership__created_date | timestamptz | Time that the Membership was created |
| membership__modified_date | timestamptz | Time that the Membership was last updated |
| membership__permission_tier | varchar | The permission tier of the User in the Organization. One of: `ADMIN`, `ORGANIZER`, `TRUSTED_HOST`, `HOST` |
| membership__organization_id | integer | Foreign Key to the Organization this User's Membership is for |
| globally_blocked_date | timestamptz | Time that the User was globally blocked from all Mobilize organizations |

## Affiliations

The `affiliations` view contains information about users who have a relationship with the Organization. In addition to volunteers who have signed up for the organization's events (who are visible in the Participations view), this view also includes users who have become affiliated through other means such as filling out the hot leads form or expressing interest in hosting an event.

| column name | type | description |
| ----------- | ---- | ----------- |
| id | integer | The Affiliation's primary key |
| organization_id | integer | The id of the current organization |
| created_date | timestamptz | Time that the Affiliation was created. |
| modified_date | timestamptz | Time that the Affiliation was last updated. |
| blocked_date | timestamptz | Timestamp that the User was blocked by the organization. `null` if the User is not blocked |
| deleted_date | timestamptz | Time that the Affiliation was deleted. `null` if the affiliation was not deleted. |
| source | varchar | The source of the user's affiliation. One of: `PARTICIPATION, HOT_LEAD, C4_FORM, PUBLIC_API, HOST_MEMBERSHIP, HOST_COMMITMENT` |
| host_commitment_source | varchar | The method the user used to commit to host events for the org. One of: `EVENT_CAMPAIGN_DISCOVERY_PAGE, SMS, EMAIL` |
| committed_to_host_date | timestamptz | The timestamp that the user committed to hosting events for the org. |
| declined_to_commit_to_host_date | timestamptz | The timestamp that the user declined to host events for the org. |
| user_id | integer | Unique identifier of the User who signed up for this participation |
| user__modified_date | timestamptz | Timestamp the User was last updated |
| user__given_name | varchar(255) | User's current first name |
| user__family_name | varchar(255) | User's current last name |
| user__email_address | citext | User's current email address |
| user__phone_number | varchar(15) | User's current phone number |
| user__locality | varchar(10) | User's city. This data is only available when collected through certain sources such as `HOT_LEAD`. |
| user__region | varchar(10) | User's state. This data is only available when collected through certain sources such as `HOT_LEAD`. |
| user__postal_code | varchar(10) | User's current zip code |
| user__globally_blocked_date | timestamptz | Timestamp that the User was globally blocked from all Mobilize organizations. `null` if the User is not blocked |


## VAN Views
The following views map roughly to VAN Event, Shift, Signup, and Person [objects](https://developers.ngpvan.com/van-api#events). The following views will only be populated if the organization has a VAN committee set.

## VAN Events

The `van_events` view contains information about VAN events that have been synced to Mobilize. Only VAN events that belong to the organization's VAN committee are included.

| column name | type | description |
| ----------- | ---- | ----------- |
| id | integer | The primary key of the VAN event |
| created_date | timestamptz | Time that the VAN event was first synced |
| modified_date | timestamptz | Time that the VAN event was last updated |
| van_id | integer | The ID of the VAN event as seen in VAN |
| committee_id | integer | The VAN committee to which this event belongs |
| event_id | integer | Non-null for normal event syncs. Foreign Key to the Event to which this event belongs |
| event_campaign_id | integer | Only non-null for GOTV event campaign syncs. Foreign Key to the EventCampaign this event is synced as part of |
| sync_aggregation | varchar | `EVENT` or `EVENT_CAMPAIGN`. Whether this is a normal event sync or a GOTV event campaign sync |


## VAN Shifts

The `van_shifts` view contains information about VAN shifts that have been synced to Mobilize. Only VAN shifts that belong to the organization's VAN committee are included.

| column name | type | description |
| ----------- | ---- | ----------- |
| id | integer | The primary key of the VAN shift |
| created_date | timestamptz | Time that the VAN shift was first synced |
| modified_date | timestamptz | Time that the VAN shift was last updated |
| van_id | integer | The ID of the VAN shift as seen in VAN |
| van_event_van_id | integer | The ID of the shift's VAN event as seen in VAN; Foreign Key to the `van_id` field in the `van_events` view |
| committee_id | integer | The VAN committee ID to which this shift belongs |
| timeslot_id | integer | Non-null for normal event syncs. Foreign Key to the Timeslot this VAN shift belongs to |
| event_campaign_id | integer | Only non-null for GOTV event campaign syncs. Foreign Key to the EventCampaign this event is synced as part of |
| start_date | timestamptz | Only non-null for GOTV event campaign syncs. Normalized start time for this VAN shift. Mapped from Timeslot start times by converting local naive time to the timezone for this event campaign |
| end_date | timestamptz | Only non-null for GOTV event campaign syncs. Normalized end time for this VAN shift. Mapped from Timeslot start times by converting local naive time to the timezone for this event campaign |
| van_event_campaign_timezone | varchar | Only non-null for GOTV event campaign syncs. Timezone for event campaign used in start and end time normalization |
| sync_aggregation | varchar | `EVENT` or `EVENT_CAMPAIGN`. Whether this is a normal event sync or a GOTV event campaign sync |

## VAN Signups

The `van_signups` view contains information about VAN signups that have been synced to Mobilize. Only VAN signups that belong to the organization's VAN committee are included.

| column name | type | description |
| ----------- | ---- | ----------- |
| id | integer | The primary key of the VAN signup |
| created_date | timestamptz | Time that the VAN signup was first synced |
| modified_date | timestamptz | Time that the VAN signup was last updated |
| timeslot_id | integer | Foreign Key to the Timeslot this VAN signup belongs to, if the signup is for an event owner and `null` otherwise |
| user_id | integer | The ID of the User associated with this VAN person; will match the `user_id` on the associated Participation |
| participation_id | integer | Foreign Key to the Participation this VAN signup belongs to, if the signup is for a participation and `null` otherwise |
| signup_type | varchar | The type of signup. One of `PARTICIPATION`, `EVENT_OWNER` |
| van_id | integer | The ID of the VAN signup as seen in VAN |
| van_event_van_id | integer | The ID of the signup's VAN event as seen in VAN; Foreign Key to the `van_id` field in the `van_events` view |
| van_shift_van_id | integer | The ID of the signup's VAN shift as seen in VAN; Foreign Key to the `van_id` field in the `van_shifts` view |
| van_person_van_id | integer | The ID of the signup's VAN person as seen in VAN; Foreign Key to the `van_id` field in the `van_persons` view |
| committee_id | integer | The VAN committee ID to which this signup belongs |

## VAN Persons

The `van_persons` view contains information about VAN persons that have been synced to Mobilize. Only VAN persons that belong to the organization's VAN committee are included.

| column name | type | description |
| ----------- | ---- | ----------- |
| id | integer | The primary key of the VAN person |
| created_date | timestamptz | Time that the VAN person was first synced |
| modified_date | timestamptz | Time that the VAN person was last updated |
| van_id | integer | The ID of the VAN person as seen in VAN |
| committee_id | integer | The VAN committee ID to which this person belongs |
| user_id | integer | The ID of the User associated with this VAN person |

## VAN Locations

The `van_locations` view contains information about the mapping of Mobilize Events to VAN locations. Only Events synced for the Organization's VAN committee are included.

| column name | type | description |
| ----------- | ---- | ----------- |
| created_date | timestamptz | Time that the VAN location was first synced |
| modified_date | timestamptz | Time that the VAN location was last updated |
| van_id | integer | The ID of the VAN location as seen in VAN |
| event_id | integer | Foreign Key to the Event this VAN location is for |
| committee_id | integer | Foreign Key to the VAN committee to which this VAN location was synced |

## Event Co-Hosts

The `event_co_hosts` view contains information about co-hosts added to events. Only co-hosts on events owned by the Organization are included.

| column name | type | description |
| ----------- | ---- | ----------- |
| id | integer | The primary key of the event-co-host relation |
| created_date | timestamptz | Time that the co-host was first added |
| modified_date | timestamptz | Time that the co-host was last updated |
| event_id | integer | The ID of the event |
| user_id | integer | The User ID of the co-host of the event |
| email | integer | The email address of the co-host of the event |

## Organizations

The `organizations` view contains information about the Organization, including the Van committee ID used for its events.

| column name | type | description |
| ----------- | ---- | ----------- |
| id | integer | The organization's primary key |
| name | varchar(100) | The public-facing name of the organization |
| slug | citext | The URL-safe string for this organization |
| committee_id | integer | The VAN committee to which this organization's events belong |

# Changelog

**2020-06-22**
- Add `registration_mode` to [`events`](#events) view

**2020-12-29**
- Fixed an issue with the [`participations`](#participations) view where the following fields were being incorrectly masked (set to null) for the event owner if the org that drove the signup was across the firewall: `event_id`, `timeslot_id`, `status`, `attended`, `experience_feedback_type`, `experience_feedback_text`

**2020-10-16**
- Add the [`event_co_hosts`](#event-co-hosts) view

**2020-09-11**
- Add the [`affiliations`](#affiliations) view
- Add `globally_blocked_at` to [`users`](#users) view
- Add `event_type_name` to [`participations`](#participations) view

**2020-07-07**
- Update [`events`](#events) view fields `organization__is_coordinated` and `organization__is_independent` to use our new database field for representing the legal compliance domain that an organization operates in.

**2020-04-08**
- Add `virtual_action_url` to [`events`](#events) view
- Add `custom_field_values` to [`participations`](#participations) view

**2020-03-17**
- Add `blocked_date` to [`users`](#users) view
- Add `user__blocked_date` to [`participations`](#participations) view

**2020-02-25**
- Add `sync_aggregation` and `event_campaign_id` to [`van_events`](#van-events) view
- Add `sync_aggregation`, `event_campaign_id`, `start_date`, `end_date`, and `van_event_campaign_timezone` to [`van_shifts`](#van-shifts) view

**2020-02-04**
- Add `van_name` to [`events`](#events) view
- Add [`users`](#users) and [`van_locations`](#van-locations) view

**2019-10-15**
- Add `accessibility_notes`, `accessibility_status`, `is_virtual`, `event_campaign_id`, and `event_campaign__slug` to [`events`](#events) view

**2019-09-19**
- Add [`van_events`](#van-events), [`van_shifts`](#van-shifts), [`van_signups`](#van-signups), and [`van_persons`](#van-persons) views

**2019-08-19**
- Add [`event_tags`](#event-tags) view
- Fix a bug where some timeslots for promoted events that had not yet been approved were included in the [`timeslots`](#timeslots) view

**2019-07-22**
- Add `DEBATE_WATCH_PARTY` as a possible event type in `events` view

**2019-07-16**
- Add `location__country` to `events` view

**2019-06-11**
- Clarify cross-firewall behavior on `participations` view for `attended`, `experience_feedback_type`, `participation_status` columns

**2019-05-30**
- Add `sms_opt_ins` view

**2019-05-14**
- Add `max_attendees` to `timeslots` view

**2022-08-04**
- Add `organizations` view

