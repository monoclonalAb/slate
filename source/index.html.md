---
title: API Reference

language_tabs: # must be one of https://github.com/rouge-ruby/rouge/wiki/List-of-supported-languages-and-lexers

toc_footers:
  - <a href='#'>Sign Up for a Developer Key</a>
  - <a href='https://github.com/slatedocs/slate'>Documentation Powered by Slate</a>

includes:
  - errors

search: true

code_clipboard: true

meta:
  - name: description
    content: Documentation for the University of Auckland Badminton Club Booking Page
---

# Introduction

> The right section will contain all the information about *Body* and *Header* parameters

Welcome to the UABC API! The documentation aims to help current and future developers understand all the endpoints present.


<!-- # Authentication

> To authorize, use this code:

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
```

> Make sure to replace `meowmeowmeow` with your API key.

Kittn uses API keys to allow access to the API. You can register a new Kittn API key at our [developer portal](http://example.com/developers).

Kittn expects for the API key to be included in all API requests to the server in a header that looks like the following:

`Authorization: meowmeowmeow`

<aside class="notice">
You must replace <code>meowmeowmeow</code> with your personal API key.
</aside> -->

# Authentication

## Validate Email

This endpoint takes in a JSON file containing an email address, and consequently sends to the address an email containing a verification code that expires with 3 minutes. This endpoint is usually used in conjuction with the [register](#register) endpoint.

> **Request**:

> - Body:

```json
  {
    "email": "email@address.com"
  }
```

### HTTP Request

`POST https://wdcc-uabc-staging.fly.dev/api/auth/register/validate-email`

### Errors

Status Code | Error Code | Message
--------- | ------- | -----------
204 | No Content | null
400 | Bad Request | multiple possible messages e.g. "Email already in use", "Invalid email address"
500 | Internal Server Error | Internal server error

## Resend Verification Code

> **Request**:

> - Body:

```json
  {
    "email": "email@address.com"
  }
```

This endpoint shares similarities with the [validate email](#validate-email) endpoint. It is used after the validate email endpoint, if users need the verification code resent to their email address. Only difference is that this endpoint accounts for rate limiting if the user sends too many requests.

### HTTP Request

`POST https://wdcc-uabc-staging.fly.dev/api/auth/register/resend-code`

### Errors

Status Code | Error Code | Message
--------- | ------- | -----------
204 | No Content | null
400 | Bad Request | multiple possible messages e.g. "Email already in use", "Invalid email address"
429 | Too Many Requests | Rate limit exceeded
500 | Internal Server Error | Internal server error

## Register

> **Request**:

> - Body:

```json
  {
    "email": "email@address.com"
    "password": "Number1Password"
    "token": "123456"
  }
```

This endpoint registers new users using a unique email address, password and the verification token sent to their email address via the [validate email](#validate-email) endpoint or [resend verification code](#resend-verification-code) endpoint.

### HTTP Request

`POST https://wdcc-uabc-staging.fly.dev/api/auth/register`

### Errors

Status Code | Error Code | Message
--------- | ------- | -----------
200 | OK | User registered successfully
400 | Bad Request | multiple possible messages e.g. "Invalid token", "Invalid email address"
500 | Internal Server Error | Internal server error

# Bookings

## Retrieve Booking


> **Request**:

> - Header:

```plaintext
  session: next-auth.session-token
```

This endpoint matches the ID of a session booking and returns a JSON file containing information related to it as long as your next-auth session token is valid and the user ID in the session booking matches your own user ID.

### HTTP Request

`GET https://wdcc-uabc-staging.fly.dev/api/bookings/<bookingID>`

### URL Parameters

Parameter | Description
--------- | -----------
bookingID | A string obfuscation of the numerical ID related to a session booking

### Errors

Status Code | Error Code | Message
--------- | ------- | -----------
400 | Bad Request | multiple possible messages e.g. "Invalid bookingId"
401 | Unauthorized | Unauthorized request
403 | Forbidden | Invalid permissions
404 | Not Found | Booking not found
500 | Internal Server Error | Internal server error

## Session Booking

> **Request**:

> - Header:

```header
  session: next-auth.session-token
```

> - Body: (for casuals)

```json
  [
    {
        "gameSessionId": 1,
        "playLevel": "beginner"
    }
  ]
```

> - Body: (for members)

```json
  [
    {
        "gameSessionId": 1,
        "playLevel": "beginner"
    },
    {
        "gameSessionId": 2,
        "playLevel": "intermediate"
    }
  ]
```

> **Response**

> - Body:

```json

```


This endpoint allows users to make a session booking as long as none of the sessions have hit maximum capacity. Members of UABC can book 2 sessions, while casuals can only book 1. Returns a JSON containing the obfuscated BookingID.

### HTTP Request

`POST https://wdcc-uabc-staging.fly.dev/api/bookings`

### Errors

Status Code | Error Code | Message
--------- | ------- | -----------
201 | Created | null
400 | Bad Request | multiple possible messages e.g. "Insufficient prepaid sessions, Duplicate game session ids"
401 | Unauthorized | Unauthorized request 
403 | Forbidden | Unverified member
404 | Not Found | User not found
409 | Conflict | Game session at max capacity
500 | Internal Server Error | Internal server error

# Cron

## Generate Sessions (Cron Job) 

> **Request**:

> - Header:

```header
  session: next-auth.session-token
  x-api-key: CRON key used to secure APIs
```

> **Response**:

> - Body:

```json
  [
    {
      "id": 170,
      "gameSessionScheduleId": 346,
      "bookingPeriodId": 31,
      "date": "2024-08-06",
      "startTime": "18:00:00",
      "endTime": "20:00:00",
      "locationName": "Kings College",
      "locationAddress": "123 University Road, Auckland",
      "memberCapacity": 40,
      "casualCapacity": 5
    },
    {
      "id": 171,
      "gameSessionScheduleId": 347,
      "bookingPeriodId": 31,
      "date": "2024-08-07",
      "startTime": "18:00:00",
      "endTime": "20:00:00",
      "locationName": "Venue Name",
      "locationAddress": "Address",
      "memberCapacity": 40,
      "casualCapacity": 5
    },
    {
      "id": 172,
      "gameSessionScheduleId": 351,
      "bookingPeriodId": 31,
      "date": "2024-08-08",
      "startTime": "19:30:00",
      "endTime": "22:00:00",
      "locationName": "UoA Sports Center",
      "locationAddress": "7 Wynyard Street, Auckland CBD",
      "memberCapacity": 20,
      "casualCapacity": 3
    }
  ]
```

This endpoint is automatically called by Amazon EventBridge Scheduler, allowing it to populate the database with game-sessions inside of an active semester ahead of time. Returns a JSON containing data on all the sessions that have been created.

### HTTP Request

`POST https://wdcc-uabc-staging.fly.dev/api/cron/generate-sessions`

### Errors

Status Code | Error Code | Message
--------- | ------- | -----------
200 | Ok | No game sessions to insert
201 | Created | null
400 | Bad Request | No active semester found
401 | Unauthorized | Missing or invalid API key
500 | Internal Server Error | Internal server error


# Game Sessions

## Get Game Session Info

> **Request**:

> - Header:

```header
  session: next-auth.session-token
```

> **Response**:

> Body:

```json
  {
    "id": 156,
    "gameSessionScheduleId": 346,
    "bookingPeriodId": 29,
    "date": "2024-07-23",
    "startTime": "18:00:00",
    "endTime": "20:00:00",
    "locationName": "Kings College",
    "locationAddress": "123 University Road, Auckland",
    "memberCapacity": 1,
    "casualCapacity": 5
  }
```

This endpoint allows users to get all the relevant information of any game session with a corresponding gameSessionId.

### HTTP Request

`GET https://wdcc-uabc-staging.fly.dev/api/game-sessions/<gameSessionId>`

### URL Parameters

Parameter | Description
--------- | -----------
gameSessionID | Integer representation of gameSessionID

### Errors

Status Code | Error Code | Message
--------- | ------- | -----------
204 | No Content | null
400 | Bad Request | multiple possible messages e.g. "Invalid id provided in the request", "Start time must be less than end time"
404 | Not Found | No Game Session found with id: `${gameSessionId}`
500 | Internal Server Error | Internal server error

## Edit Existing Game Session

> **Request**:

> - Header:

```header
  session: next-auth.session-token (needs admin role)
```

> - Body:

```json
  {
    "id": 170,
    "gameSessionScheduleId": 346,
    "bookingPeriodId": 31,
    "date": "2024-08-06",
    "startTime": "19:00:00",
    "endTime": "20:00:00",
    "locationName": "Kings College",
    "locationAddress": "123 University Road, Auckland",
    "memberCapacity": 40,
    "casualCapacity": 5
  }
```

This endpoint allows admins to update an existing game session.

### HTTP Request

`PUT https://wdcc-uabc-staging.fly.dev/api/game-sessions/<gameSessionId>`

### URL Parameters

Parameter | Description
--------- | -----------
gameSessionID | Integer representation of gameSessionID

### Errors

Status Code | Error Code | Message
--------- | ------- | -----------
204 | No Content | null
400 | Bad Request | multiple possible messages e.g. "Invalid id provided in the request", "Start time must be less than end time"
403 | Forbidden | Access denied
404 | Not Found | No Game Session found with id: `${gameSessionId}`
500 | Internal Server Error | Internal server error

## Delete Game Session

> **Request**:

> - Header:

```header
  session: next-auth.session-token (needs admin role)
```

This endpoint allows admins to delete existing game sessions.

### HTTP Request

`DELETE https://wdcc-uabc-staging.fly.dev/api/game-sessions/<gameSessionId>`

### URL Parameters

Parameter | Description
--------- | -----------
gameSessionID | Integer representation of gameSessionID

### Errors

Status Code | Error Code | Message
--------- | ------- | -----------
204 | No Content | null
400 | Bad Request | Invalid id provided in the request
404 | Not Found | Game session with id `${gameSessionId}` does not exist.
500 | Internal Server Error | Internal server error

## Download Attendees List 

> **Request**:

> - Header:

```header
  session: next-auth.session-token (needs admin role)
```

This endpoint allows admins to download the list of all attendees of a session as a CSV. The CSV contains details of their first name, last name, email, play level, and if they are a member/pro or not. 

### HTTP Request

`GET https://wdcc-uabc-staging.fly.dev/api/game-sessions/<gameSessionId>/download`

### URL Parameters

Parameter | Description
--------- | -----------
gameSessionID | Integer representation of gameSessionID

### Errors

Status Code | Error Code | Message
--------- | ------- | -----------
200 | Ok | null
400 | Bad Request | Invalid id provided in the request
401 | Unauthorized | ERROR: Unauthorized request
403 | Forbidden | ERROR: No valid permissions
404 | Not Found | No Game Session found with id: `${gameSessionId}`
500 | Internal Server Error | Internal server error

## Get Active Dates (Dates with Game Sessions)

> **Request**:

> - Header:

```header
  session: next-auth.session-token (needs admin role)
```

> **Response**:

> - Body (between `?start-date=2024-08-25&end-date=2024-10-07`):

```json
  [
    "2024-09-11",
    "2024-09-12",
    "2024-09-17",
    "2024-09-18",
    "2024-09-19",
    "2024-09-24",
    "2024-09-25",
    "2024-09-26"
  ] 
```

This endpoint gets all the active dates with game sessions between a given start and end date. It returns the string dates in the format "yyyy-mm-dd" in a list.

### HTTP Request

`GET https://wdcc-uabc-staging.fly.dev/api/game-sessions/active-dates`

### Query Parameters

Query | Description
--------- | -----------
start-date | starting date in the format yyyy-mm-dd
end-date | ending date in the format yyyy-mm-dd

### Errors

Status Code | Error Code | Message
--------- | ------- | -----------
200 | Ok | null
400 | Bad Request | Bad Request
401 | Unauthorized | ERROR: Unauthorized request
403 | Forbidden | ERROR: No valid permissions

## Get Current Game Sessions 

> **Response**:

> - Body:

```json
  [
    {
      "id": 167,
      "date": "2024-07-30",
      "startTime": "18:00:00",
      "endTime": "19:00:00",
      "locationName": "Kings College",
      "locationAddress": "123 University Road, Auckland",
      "memberCapacity": 40,
      "casualCapacity": 5,
      "memberBookingCount": 1,
      "casualBookingCount": 0
    },
    {
      "id": 168,
      "date": "2024-07-31",
      "startTime": "18:00:00",
      "endTime": "20:00:00",
      "locationName": "Kings College",
      "locationAddress": "123 University Road, Auckland",
      "memberCapacity": 40,
      "casualCapacity": 5,
      "memberBookingCount": 0,
      "casualBookingCount": 0
    },
    {
      "id": 169,
      "date": "2024-08-01",
      "startTime": "19:30:00",
      "endTime": "22:00:00",
      "locationName": "UoA Sports Center",
      "locationAddress": "7 Wynyard Street, Auckland CBD",
      "memberCapacity": 20,
      "casualCapacity": 3,
      "memberBookingCount": 0,
      "casualBookingCount": 0
    }
  ]
```

This endpoint gets all the sessions which you can currently book a spot for. Returns a list of JSON's containing each sessions information.

### HTTP Request

`GET https://wdcc-uabc-staging.fly.dev/api/game-sessions/current`

### Errors

Status Code | Error Code | Message
--------- | ------- | -----------
200 | Ok | null
500 | Internal Server Error | Internal Server Error: `${error}`

## Get Game Session Information for a specific date

> **Request**:

> - Header:

```header
  session: next-auth.session-token (needs admin role)
```

> **Response**:

> - Body (if there does not exist a game session):

```json
  {
    "exists": false,
    "canCreate": true,
    "message": "No game session found for this date",
    "data": {
      "semesterName": "Semester 2",
      "bookingOpen": "2024-08-05T00:00:00.000Z"
    }
  }
```

> - Body (if there exists a game session)

```json
  {
    "exists": true,
    "canCreate": true,
    "data": {
      "id": 171,
      "gameSessionScheduleId": 347,
      "bookingPeriodId": 31,
      "date": "2024-08-07",
      "startTime": "18:00:00",
      "endTime": "20:00:00",
      "locationName": "Venue Name",
      "locationAddress": "Address",
      "memberCapacity": 40,
      "casualCapacity": 5,
      "bookingOpen": "2024-08-05T00:00:00.000Z",
      "attendees": 0
    }
  }
```

This endpoint allows admins to check if there exists a game session on a specific date. If there does, then the endpoint returns a JSON containing the data of the existing game session.

### HTTP Request

`GET https://wdcc-uabc-staging.fly.dev/api/game-sessions`

### Query Parameters

Query | Description
--------- | -----------
date | date in the format yyyy-mm-dd

### Errors

Status Code | Error Code | Message
--------- | ------- | -----------
200 | Ok | null
400 | Bad Request | Bad Request
401 | Unauthorized | ERROR: Unauthorized request
403 | Forbidden | ERROR: No valid permissions
404 | Not Found | multiple possible messages e.g. "No ongoing semester found for this date", "No game session found for this date"
500 | Internal Server Error | Internal server error

# Schedules

## Get Schedule Data

> **Response**:

> - Body:

```json
  {
    "id": 346,
    "semesterId": 137,
    "weekday": "Tuesday",
    "startTime": "18:00:00",
    "endTime": "20:00:00",
    "locationName": "Kings College",
    "locationAddress": "123 University Road, Auckland",
    "memberCapacity": 40,
    "casualCapacity": 5
  }
```

This endpoint allows users to get all the relevant information of any scheduled game session in a semester with the corresponding scheduleId.

### HTTP Request

`GET https://wdcc-uabc-staging.fly.dev/api/schedules/<scheduleId>`

### Query Parameters

Query | Description
--------- | -----------
scheduleId | Integer representation of scheduleId

### Errors

Status Code | Error Code | Message
--------- | ------- | -----------
200 | Ok | null
404 | Not Found | No GameSessionSchedule found for id: `${scheduleId}`
500 | Internal Server Error | Internal server error

## Edit Schedule Data

> **Request**:

> - Header:

```plaintext
  session: next-auth.session-token (needs admin role)
```

> - Body:

```json
  {
    "id": 346,
    "semesterId": 137,
    "weekday": "Tuesday",
    "startTime": "19:00:00",
    "endTime": "20:00:00",
    "locationName": "Kings College",
    "locationAddress": "123 University Road, Auckland",
    "memberCapacity": 40,
    "casualCapacity": 5
  }
```

> **Response**:

> - Body:

```json
  [
    {
      "id": 346,
      "semesterId": 137,
      "weekday": "Tuesday",
      "startTime": "19:00:00",
      "endTime": "20:00:00",
      "locationName": "Kings College",
      "locationAddress": "123 University Road, Auckland",
      "memberCapacity": 40,
      "casualCapacity": 5
    }
  ]
```

This endpoint allows admins to edit relevant information of any scheduled game session in a semester with the corresponding scheduleId.

### HTTP Request

`PUT https://wdcc-uabc-staging.fly.dev/api/schedules/<scheduleId>`

### Query Parameters

Query | Description
--------- | -----------
scheduleId | Integer representation of scheduleId

### Errors

Status Code | Error Code | Message
--------- | ------- | -----------
200 | Ok | null
400 | Bad Request | multiple possible messages e.g. "Start time must be before end time", "No GameSessionSchedule found for id: `scheduleId`"
401 | Unauthorized | ERROR: Unauthorized request
403 | Forbidden | ERROR: No valid permissions
500 | Internal Server Error | Internal server error

## Delete Schedule Data

> **Request**:

> - Header:

```plaintext
  session: next-auth.session-token (needs admin role)
```

```json
  {
    "id": 346,
    "semesterId": 137,
    "weekday": "Tuesday",
    "startTime": "19:00:00",
    "endTime": "20:00:00",
    "locationName": "Kings College",
    "locationAddress": "123 University Road, Auckland",
    "memberCapacity": 40,
    "casualCapacity": 5
  }
```

> **Response**:

> - Body:

```json
  [
    {
      "id": 346,
      "semesterId": 137,
      "weekday": "Tuesday",
      "startTime": "19:00:00",
      "endTime": "20:00:00",
      "locationName": "Kings College",
      "locationAddress": "123 University Road, Auckland",
      "memberCapacity": 40,
      "casualCapacity": 5
    }
  ]
```

This endpoint allows admins to delete any scheduled game session in a semester with the corresponding scheduleId.

### HTTP Request

`DELETE https://wdcc-uabc-staging.fly.dev/api/schedules/<scheduleId>`

### Query Parameters

Query | Description
--------- | -----------
scheduleId | Integer representation of scheduleId

### Errors

Status Code | Error Code | Message
--------- | ------- | -----------
204 | No Content | null
400 | Bad Request | No GameSessionSchedule found for id: `${scheduleId}`
401 | Unauthorized | ERROR: Unauthorized request
403 | Forbidden | ERROR: No valid permissions
500 | Internal Server Error | Internal server error

# Semesters

## Get All Semester Information

> **Request**:

> - Header:

```plaintext
  session: next-auth.session-token (needs admin role)
```

> **Response**:

> - Body:

```json
  [
    {
      "id": 137,
      "name": "Semester 2",
      "startDate": "2024-07-15",
      "endDate": "2024-11-11",
      "breakStart": "2024-08-26",
      "breakEnd": "2024-09-06",
      "bookingOpenDay": "Monday",
      "bookingOpenTime": "12:00:00",
      "createdAt": "2024-07-19T01:33:44.692Z"
    },
    {
      "id": 138,
      "name": "Semester 1",
      "startDate": "2025-02-26",
      "endDate": "2025-06-24",
      "breakStart": "2025-03-29",
      "breakEnd": "2025-04-12",
      "bookingOpenDay": "Monday",
      "bookingOpenTime": "12:00:00",
      "createdAt": "2024-07-21T10:58:38.591Z"
    }
  ]
```

This endpoint allows admins to get all the relevant information about all the semesters that exist right now.

### HTTP Request

`GET https://wdcc-uabc-staging.fly.dev/api/semesters`

### Errors

Status Code | Error Code | Message
--------- | ------- | -----------
200 | Ok | null
401 | Unauthorized | ERROR: Unauthorized request
403 | Forbidden | ERROR: No valid permissions
500 | Internal Server Error | Internal server error

## Get Specific Semester Information

> **Request**:

> - Header:

```plaintext
  session: next-auth.session-token (needs admin role)
```

> **Response**:

> - Body:

```json
  {
    "id": 137,
    "name": "Semester 2",
    "startDate": "2024-07-15",
    "endDate": "2024-11-11",
    "breakStart": "2024-08-26",
    "breakEnd": "2024-09-06",
    "bookingOpenDay": "Monday",
    "bookingOpenTime": "12:00:00",
    "createdAt": "2024-07-19T01:33:44.692Z"
  }
```

This endpoint allows admins to get all the relevant information of any semester with the corresponding semesterId. 

### HTTP Request

`GET https://wdcc-uabc-staging.fly.dev/api/semesters/<semesterId>`

### Query Parameters

Query | Description
--------- | -----------
semesterId | Integer representation of semesterId

### Errors

Status Code | Error Code | Message
--------- | ------- | -----------
200 | Ok | null
401 | Unauthorized | ERROR: Unauthorized request
403 | Forbidden | ERROR: No valid permissions
404 | Not Found Error | No semester found for id: `${semesterId}`
500 | Internal Server Error | Internal server error

## Delete Existing Semester

> **Request**:

> - Header:

```plaintext
  session: next-auth.session-token (needs admin role)
```

This endpoint allows admins to delete the semester with the corresponding semesterId.

### HTTP Request

`DELETE https://wdcc-uabc-staging.fly.dev/api/semesters/<semesterId>`

### Query Parameters

Query | Description
--------- | -----------
semesterId | Integer representation of semesterId

### Errors

Status Code | Error Code | Message
--------- | ------- | -----------
204 | No Content | null
401 | Unauthorized | ERROR: Unauthorized request
403 | Forbidden | ERROR: No valid permissions
404 | Not Found | No semester found for id: ${semesterId}``
500 | Internal Server Error | Internal server error

## Edit Existing Semester

> **Request**:

> - Header:

```plaintext
  session: next-auth.session-token (needs admin role)
```

> - Body:

```json
  {
    "name": "Semester 3",
    "startDate": "2026-01-01",
    "endDate": "2026-12-31",
    "breakStart": "2026-05-01",
    "breakEnd": "2026-05-31",
    "bookingOpenDay": "Monday",
    "bookingOpenTime": "01:23:00",
    "createdAt": "2024-08-10T09:11:46.366Z"
  }
```

This endpoint allows admins to edit the semester with the corresponding semesterId.

### HTTP Request

`PUT https://wdcc-uabc-staging.fly.dev/api/semesters/<semesterId>`

### Query Parameters

Query | Description
--------- | -----------
semesterId | Integer representation of semesterId

### Errors

Status Code | Error Code | Message
--------- | ------- | -----------
204 | No Content | null
400 | Bad Request | multiple possible messages e.g. "Start date must be less than end date", "Semesters cannot overlap"
401 | Unauthorized | ERROR: Unauthorized request
403 | Forbidden | ERROR: No valid permissions
500 | Internal Server Error | Internal server error

## Create New Semester

> **Request**:

> - Header:

```plaintext
  session: next-auth.session-token (needs admin role)
```

> - Body:

```json
  {
    "name": "Semester 3",
    "startDate": "2026-01-01",
    "endDate": "2026-12-31",
    "breakStart": "2026-05-01",
    "breakEnd": "2026-05-31",
    "bookingOpenDay": "Sunday",
    "bookingOpenTime": "01:23:00"
  }
```

> **Response**:

> - Body:

```json
  [
    {
      "id": 142,
      "name": "Semester 3",
      "startDate": "2026-01-01",
      "endDate": "2026-12-31",
      "breakStart": "2026-05-01",
      "breakEnd": "2026-05-31",
      "bookingOpenDay": "Sunday",
      "bookingOpenTime": "01:23:00",
      "createdAt": "2024-08-10T07:20:33.714Z"
    }
  ]
```

This endpoint allows admins to create a new Semester.

### HTTP Request

`PUT https://wdcc-uabc-staging.fly.dev/api/semesters`

### Errors

Status Code | Error Code | Message
--------- | ------- | -----------
204 | No Content | null
401 | Unauthorized | ERROR: Unauthorized request
403 | Forbidden | ERROR: No valid permissions
500 | Internal Server Error | Internal server error

## Get All Schedules Related to Existing Semester

> **Request**:

> - Header:

```plaintext
  session: next-auth.session-token
```

> **Response**:

> - Body:

```json
  [
    {
      "id": 358,
      "semesterId": 137,
      "weekday": "Wednesday",
      "startTime": "17:00:00",
      "endTime": "19:00:00",
      "locationName": "Auckland Badminton Association",
      "locationAddress": "99 Gillies Avenue, Epsom",
      "memberCapacity": 40,
      "casualCapacity": 5
    },
    {
      "id": 359,
      "semesterId": 137,
      "weekday": "Thursday",
      "startTime": "19:30:00",
      "endTime": "22:00:00",
      "locationName": "Kings School",
      "locationAddress": "258 Remuera Road, Remuera",
      "memberCapacity": 40,
      "casualCapacity": 5
    },
    {
      "id": 360,
      "semesterId": 137,
      "weekday": "Friday",
      "startTime": "17:00:00",
      "endTime": "19:00:00",
      "locationName": "Auckland Badminton Association",
      "locationAddress": "99 Gillies Avenue, Epsom",
      "memberCapacity": 40,
      "casualCapacity": 5
    }
  ]
```

This endpoint allows users to get all the schedules inside the semester with the corresponding semesterId.

### HTTP Request

`GET https://wdcc-uabc-staging.fly.dev/api/semesters/<semesterId>/schedules`

### Query Parameters

Query | Description
--------- | -----------
semesterId | Integer representation of semesterId

### Errors

Status Code | Error Code | Message
--------- | ------- | -----------
200 | Ok | null
500 | Internal Server Error | Internal server error

## Create New Schedule in an Existing Semester

> **Request**:

> - Header:

```plaintext
  session: next-auth.session-token (needs admin role)
```

> - Body:

```json
  {
    "weekday": "Sunday",
    "startTime": "01:23:00",
    "endTime": "04:56:00",
    "locationName": "Home",
    "locationAddress": "My Bed",
    "memberCapacity": 50,
    "casualCapacity": 50
  }
```

> **Response**:

> - Body:

```json
  [
    {
      "id": 366,
      "semesterId": 143,
      "weekday": "Sunday",
      "startTime": "01:23:00",
      "endTime": "04:56:00",
      "locationName": "Home",
      "locationAddress": "My Bed",
      "memberCapacity": 50,
      "casualCapacity": 50
    }
  ]
```

This endpoint allows admins to create a schedule inside the semester with the corresponding semesterId.

### HTTP Request

`GET https://wdcc-uabc-staging.fly.dev/api/semesters/<semesterId>/schedules`

### Query Parameters

Query | Description
--------- | -----------
semesterId | Integer representation of semesterId

### Errors

Status Code | Error Code | Message
--------- | ------- | -----------
201 | Created | null
401 | Unauthorized | ERROR: Unauthorized request
403 | Forbidden | ERROR: No valid permissions
500 | Internal Server Error | Internal server error

# Users

## Get all Users

> **Request**:

> - Header:

```plaintext
  session: next-auth.session-token (needs admin role)
```

> **Response**:

> - Body:

```json
  [
    {
      "id": "9b0324c4-862c-48e5-8ce6-b470d4629860",
      "firstName": "Eric",
      "lastName": "Zheng",
      "email": "airwreck@gmail.com"
    },
    {
      "id": "a203e353-fd23-444d-91ba-27c4a036e8d0",
      "firstName": "Darrell",
      "lastName": "Herzog",
      "email": "Ike.Mills22@yahoo.com"
    }
  ]
```

This endpoint allows admins to get a JSON containing the first name, last name, email address and unique ID of every member that has created an account.

### HTTP Request

`GET https://wdcc-uabc-staging.fly.dev/api/users`

### Query Parameters

Query | Description
--------- | -----------
semesterId | Integer representation of semesterId

### Errors

Status Code | Error Code | Message
--------- | ------- | -----------
200 | Ok | null
403 | Forbidden | ERROR: No valid permissions
500 | Internal Server Error | Internal server error

## Get Specific User

> **Request**:

> - Header:

```plaintext
  session: next-auth.session-token (needs admin role)
```

> **Response**:

> - Body:

```json
  {
    "id": "9b0324c4-862c-48e5-8ce6-b470d4629860",
    "firstName": "Eric",
    "lastName": "Zheng",
    "email": "airwreck@gmail.com",
    "member": false,
    "verified": false,
    "prepaidSessions": 0
  }
```

This endpoint allows admins to get all the relevant information related to a user using their unique userId.

### HTTP Request

`GET https://wdcc-uabc-staging.fly.dev/api/users/<userId>`

### Query Parameters

Query | Description
--------- | -----------
userId | String representation of userId

### Errors

Status Code | Error Code | Message
--------- | ------- | -----------
200 | Ok | null
401 | Unauthorized | ERROR: Unauthorized request
403 | Forbidden | ERROR: No valid permissions
404 | Not Found | No User found for id: `${userId}`
500 | Internal Server Error | Internal server error

## Delete Specific User

> **Request**:

> - Header:

```plaintext
  session: next-auth.session-token (needs admin role)
```

This endpoint allows admins to delete a user using their unique userId.

### HTTP Request

`DELETE https://wdcc-uabc-staging.fly.dev/api/users/<userId>`

### Query Parameters

Query | Description
--------- | -----------
userId | String representation of userId

### Errors

Status Code | Error Code | Message
--------- | ------- | -----------
204 | No Content | null
401 | Unauthorized | ERROR: Unauthorized request
403 | Forbidden | ERROR: No valid permissions
404 | Not Found | No User found for id: `${userId}`
500 | Internal Server Error | Internal server error

## Onboarding User

> **Request**:

> - Header:

```plaintext
  session: next-auth.session-token (needs to be the session token the user `${userId}` has)
```

> - Body:

```json
  {
    "firstName": "Eric",
    "lastName": "Zheng",
    "member": true
  }
```

This endpoint allows users to onboard themself without having to go through the long and tedious onboarding process.

### HTTP Request

`PATCH https://wdcc-uabc-staging.fly.dev/api/users/<userId>/onboard`

### Query Parameters

Query | Description
--------- | -----------
userId | String representation of userId

### Errors

Status Code | Error Code | Message
--------- | ------- | -----------
204 | No Content | null
400 | Bad Request | multiple possible messages  
401 | Unauthorized | ERROR: Unauthorized request
403 | Forbidden | ERROR: No valid permissions
404 | Not Found | No User found for id: `${userId}`
500 | Internal Server Error | Internal server error

## Approve User Membership:

> **Request**:

> - Header:

```plaintext
  session: next-auth.session-token (needs admin role)
```

> - Body:

```json
  {
    "prepaidSessions": 74
  }
```

This endpoint allows admins to approve a user's membership and allocates them the pre-paid sessions that they have paid for. If the endpoint call is successful, the "verified" column of the user will be toggled to `TRUE`.

### HTTP Request

`PATCH https://wdcc-uabc-staging.fly.dev/api/users/<userId>/membership/approve`

### Query Parameters

Query | Description
--------- | -----------
userId | String representation of userId

### Errors

Status Code | Error Code | Message
--------- | ------- | -----------
204 | No Content | null
401 | Unauthorized | ERROR: Unauthorized request
403 | Forbidden | ERROR: No valid permissions
404 | Not Found | No User found for id: `${userId}`
500 | Internal Server Error | Internal server error

## Reject User Membership:

> **Request**:

> - Header:

```plaintext
  session: next-auth.session-token (needs admin role)
```

> - Body:

```json
  {
    "prepaidSessions": 74
  }
```

This endpoint allows admins to reject a user's membership. If the endpoint call is successful, the "member" column of the user will be toggled to `FALSE`.

### HTTP Request

`PATCH https://wdcc-uabc-staging.fly.dev/api/users/<userId>/membership/approve`

### Query Parameters

Query | Description
--------- | -----------
userId | String representation of userId

### Errors

Status Code | Error Code | Message
--------- | ------- | -----------
204 | No Content | null
401 | Unauthorized | ERROR: Unauthorized request
403 | Forbidden | ERROR: No valid permissions
404 | Not Found | No User found for id: `${userId}`
500 | Internal Server Error | Internal server error