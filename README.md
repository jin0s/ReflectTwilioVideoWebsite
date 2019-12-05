# Twilio Web Application To Connect To Reflect Mirror

## Quick Start

Within a terminal at the parent directory run the following commands to start the webserver.

```
npm install

node server
Express server running on *:5000

http//:localhost:5000
```

Then open a browser and navigate to http://localhost:5000 and type in **roomtest** as the room.

For demo purposes, we disabled grabing the room name from the server and just set the default room to "roomtest"

If you want to dynamically create the rooms on the database side and store them on the table TwilioRooms, modify the CreateAppointment stored procedure on the database. We suggest hashing to room name or use UUID to have a secure. Right now it is using concatenation of the instructorid, classname, and start_datetime.

---

## Design Diagrams

### System Diagram

![alt text](docs/DeploymentDiagram.png)

### Database Diagram

![alt text](docs/ERDiagram.png)

### Designed for Dropping Classes

The ScheduledAppointments Table is designed for dropping classes. As users drops classes a the cancled column is assigned a 1. Afterwards the DeleteAppointment stored procedure will write a new row on the table with the same classname, instructor, roomid, and datetimes

![alt text](docs/canceledappointments.png)

Here appointment ids 23 and 23 are appended as appointment ids 44 and 45
![alt text](docs/remadeappointments.png)

---

## API List

GET: **/appointment?instructorid=1&date=2019-12-04**

Lambda Function: [ReflectGetAllAppointmentsByDate](https://us-east-2.console.aws.amazon.com/lambda/home?region=us-east-2#/functions/ReflectGetAllAppointmentsByDate?tab=configuration)

_returns_

```json
[
  {
    "appointment_id": 43,
    "name": "callstored",
    "instructor_id": 1,
    "start_datetime": "2019-12-12T00:00:00.000Z",
    "end_datetime": "2019-12-12T00:00:00.000Z",
    "description": "callstored"
  }
]
```

POST: **/appointment**

Lambda Function: [ReflectCreateAppointment](https://us-east-2.console.aws.amazon.com/lambda/home?region=us-east-2#/functions/ReflectCreateAppointment)

_send post body_

```json
{
  "classname": "Go Knights! Charge On",
  "instructor_id": "1",
  "start_datetime": "2019-12-30 00:50:00",
  "end_datetime": "2019-12-30 01:50:00",
  "description_text": "This workout will have you sweating!"
}
```

_returns_

```json
{
  "fieldCount": 0,
  "affectedRows": 1,
  "insertId": 0,
  "info": "",
  "serverStatus": 2,
  "warningStatus": 0
}
```

DELETE: **/appointment**

Lambda Function: [ReflectDeleteAppointment](https://us-east-2.console.aws.amazon.com/lambda/home?region=us-east-2#/functions/ReflectDeleteAppointment)

_send post body_

```json
{
  "appointment_id": "22"
}
```

_returns_

```json
{
  "status": "SUCCESS",
  "info": {
    "fieldCount": 0,
    "affectedRows": 0,
    "insertId": 0,
    "info": "",
    "serverStatus": 2,
    "warningStatus": 0
  }
}
```

PUT: **/appointment**

Lambda Function: [ReflectSignupForAppointment](https://us-east-2.console.aws.amazon.com/lambda/home?region=us-east-2#/functions/ReflectSignupForAppointment)

_send post body_

```json
{
  "customer_id": "1",
  "appointment_id": "22"
}
```

_returns_

```json
{
  "status": "SUCCESS"
}
```

POST: **/appointment/user**

Lambda Function: [ReflectGetAllUserAppointment](https://us-east-2.console.aws.amazon.com/lambda/home?region=us-east-2#/functions/ReflectGetAllUserAppointment)

_returns_

```json
{
  "customer_id": "31",
  "end_datetime": "2019-12-03 07:41:00"
}
```

_returns_

```json
[
{
  "appointment_id": 47,
  "classname": "Ready to Rock?",
  "customer_id": 31,
  "instructor_id": 1,
  "room_id": 22,
  "start_datetime": "2019-12-03T12:00:00.000Z",
  "end_datetime": "2019-12-03T13:00:00.000Z",
  "description": "This is the best description for a class ever Enter Description Here...",
  "canceled": {
    "type": "Buffer",
    "data": [
      0
    ]
  },
  "canceled_datetime": null,
  "instructor_name": "Jaime Wilbanks",
  "instructor_image": "https://cdn.reflectgateway.com/inst/07222019031440PM-Jaime4.png"
},
{
  "appointment_id": 53,
  "classname": "Senior Demo",
  "customer_id": 31,
  "instructor_id": 2,
  "room_id": 23,
  "start_datetime": "2019-12-03T12:00:00.000Z",
  "end_datetime": "2019-12-03T13:00:00.000Z",
  "description": "We are gonna graduate :)",
  "canceled": {
    "type": "Buffer",
    "data": [
      0
    ]
  },
  "canceled_datetime": null,
  "instructor_name": "Nicole Griffin",
  "instructor_image": "https://cdn.reflectgateway.com/inst/07222019031423PM-Nicole2.png"
}, ...
]
```

POST:  **/twilio**

Lambda Function: [ReflectGetTwilioAccessToken](https://us-east-2.console.aws.amazon.com/lambda/home?region=us-east-2#/functions/ReflectGetTwilioAccessToken)

_send post body_

```json
{
  "username": "yuyuhakasho",
  "room_name": "spirit_gun"
}
```

_returns_

```json
{
  "message": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCIsImN0eSI6InR3aWxpby1mcGE7dj0xIn0.eyJqdGkiOiJTSzQ3MzczNjViYmExM2I1NWUxOTgzOWJjNzgxZDA5OTVmLTE1NzU1MTE0MDAiLCJncmFudHMiOnsiaWRlbnRpdHkiOiJ5dXl1aGFrYXNobyIsInZpZGVvIjp7InJvb20iOiJzcGlyaXRfZ3VuIn19LCJpYXQiOjE1NzU1MTE0MDAsImV4cCI6MTU3NTUxNTAwMCwiaXNzIjoiU0s0NzM3MzY1YmJhMTNiNTVlMTk4MzliYzc4MWQwOTk1ZiIsInN1YiI6IkFDMTgxYTcyZWE0YzJhYTI3ZTE5OTYxMjA3OTk3OTJkNjIifQ.eokQRYLlXp__pXV2oMdNJ0P5xVwXNtNe8V3SbBwN5l0",
  "username": "yuyuhakasho",
  "room_name": "spirit_gun"
}
```
---

## Modify ReflectGetTwilioAccessToken For Production
From your Twilio Console, you just need to grab your Account SID, API Key and API secret key to generate access token for reflect mirrors. Then go to the [ReflectGetTwilioAccessToken](https://us-east-2.console.aws.amazon.com/lambda/home?region=us-east-2#/functions/ReflectGetTwilioAccessToken)
 Lambda function and modify the Environment variables.
 
![alt text](docs/twilioconfig.png)