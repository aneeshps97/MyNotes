Here's how I would structure Rhino if the goals are:

1. Learn microservices properly.
2. Build something that could potentially become a real product.
3. Keep service boundaries clean.
4. Avoid overengineering.

# Architecture

```

                    API Gateway
                         |
 ------------------------------------------------
 |        |         |         |        |         |
Auth    User     Group   Membership  Post     Ride
Service Service  Service   Service   Service Service
                         |
                   Notification
                      Service

(Optional Later)
Payment Service
Location Service


```

---

# 1. Auth Service

### Responsibility

Authentication and authorization only.

### Entities

```

UserCredential
--------------
userId
username
email
passwordHash
enabled

RefreshToken
------------
tokenId
userId
expiry

```

### APIs

```

POST /register
POST /login
POST /refresh-token
POST /logout

```

---

# 2. User Service

### Responsibility

Personal information.

### Entities

```

UserProfile
-----------
userId
username
name
bio
profilePicture

Bike
----
bikeId
userId
brand
model
year

EmergencyContact
----------------
contactId
userId
name
phoneNumber
relationship

MedicalInfo
-----------
medicalInfoId
userId
bloodGroup
allergies
medicalConditions

```

### APIs

```
GET /users/{id}
PUT /users/{id}

GET /users/search
```

---

# 3. Group Service

### Responsibility

Group management.

### Entities

```

Group
-----
groupId
name
description
createdBy
createdDate
visibility

GroupSettings
-------------
groupId
allowJoinRequests
maxMembers

```

### APIs

```

POST /groups
GET /groups/{id}
PUT /groups/{id}
DELETE /groups/{id}


```

---

# 4. Membership Service

### Responsibility

Members, requests, invitations.

### Entities

```

GroupMember
-----------
groupId
userId
role

Role
----
ADMIN
MODERATOR
MEMBER

JoinRequest
-----------
requestId
groupId
userId
status

Invitation
----------
invitationId
groupId
userId
status


```

### APIs

```


POST /groups/{id}/request

POST /groups/{id}/approve

POST /groups/{id}/reject

POST /groups/{id}/invite


```

---

# 5. Post Service

### Responsibility

Community feed.

### Entities

```

Post
----
postId
groupId
userId
caption
createdAt

Media
-----
mediaId
postId
url
type

Like
----
likeId
postId
userId

```

### Future

```

Comment
-------
commentId
postId
userId
text

```

### APIs

```

POST /posts

GET /groups/{id}/posts

POST /posts/{id}/like

```

---

# 6. Ride Service

### Responsibility

Everything related to rides.

### Entities

```

Ride
----
rideId
groupId
createdBy

rideName
startDate
endDate

status
UPCOMING
ONGOING
COMPLETED

maxParticipants

expensePerHead

```

---

### Ride Stops

```

RideStop
--------
stopId
rideId

name
type

START
STAY
RESTAURANT
FUEL
END

latitude
longitude
sequence

```

---

### Participants

```

RideParticipant
---------------
rideId
userId

joinedDate

paymentStatus

attendanceStatus

```

---

### Attendance

```

Attendance
----------
attendanceId
rideId
userId
markedAt

```

### APIs

```

POST /rides

POST /rides/{id}/join

POST /rides/{id}/attendance

POST /rides/{id}/start

POST /rides/{id}/complete

GET /rides/{id}

```

---

# 7. Notification Service

### Responsibility

Notifications only.

### Entities

```

Notification
------------
notificationId
userId

title
message

type

readStatus
createdAt

```

### Types

```

GROUP_INVITE

GROUP_REQUEST_APPROVED

NEW_POST

NEW_RIDE

RIDE_STARTING

EMERGENCY_ALERT

```

### APIs

Mostly event-driven.

Very few direct APIs.

---

# 8. Emergency Alert Service (Optional V2)

### Responsibility

Ride alerts.

### Entities

```

EmergencyAlert
--------------
alertId
rideId
userId

type

createdAt
status

```

### Types

```
FUEL_STOP

REFRESHMENT

MECHANICAL_ISSUE

ACCIDENT

MEDICAL_EMERGENCY

```

### Flow

```
User presses button

Emergency Alert Created

Notification Service

Push everyone

```

---

# 9. Payment Service (Optional V3)

### Entities

```
Payment
-------
paymentId

rideId
userId

amount

status

transactionReference
```

---

# Sprint Plan

## Sprint 1

Infrastructure

```
Gateway
Config Server
Eureka
Docker Compose
```

Services:

```
Auth Service
User Service
```

Flutter:

```
Login
Register
Profile Setup

```

---

## Sprint 2

Services:

```
Group Service
Membership Service

```

Flutter:

```
Groups Tab

Create Group

Join Group

Join Requests

```

At the end:

```
User
Create Group
Join Group
Approve Requests

```

works.

---

## Sprint 3

Services:

```
Post Service
```

Flutter:

```
Community Feed

Create Post

Like Post

```

At the end:

```
Users can post photos.
```

---

## Sprint 4

Services:

```
Ride Service
```

Flutter:

```

Create Ride

Upcoming Rides

Ride Details

Join Ride

```

At the end:

```
Users can create and join rides.
```

---

## Sprint 5

Services:

```
Notification Service
```

Flutter:

```

Ride Alerts Tab

Notification Center

```

At the end:

```
New ride notifications work.
```

---

## Sprint 6

Features:

```
Attendance

Ride Start

Ride Complete

```

Flutter:

```
Attendance Screen

Ongoing Ride Screen

```

---

## Sprint 7

Features:

```
Emergency Alerts
```

Flutter:

```
Emergency ButtonAlert Popup
```

---

## Sprint 8

Optional Advanced Features

```

Payment Service

Kafka

Event Driven Communication

Redis

Monitoring

Tracing

```