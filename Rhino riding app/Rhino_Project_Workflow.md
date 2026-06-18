# Rhino — Updated Product Workflow & System Design

# Product Vision

Rhino is a motorcycle ride coordination and community platform designed for:
- bike groups
- ride planning
- ride coordination
- ride memories
- rider safety
- event management

The platform combines:
- social networking
- ride lifecycle management
- emergency coordination
- community interaction

into a single ecosystem.

---

# SYSTEM WORKFLOW

# 1. USER ONBOARDING FLOW

```text
User Opens App
        ↓
Signup/Login
        ↓
Create Rider Profile
        ↓
Add:
- Bike Details
- Emergency Contact
- Medical Info
- Blood Group
        ↓
User Dashboard
```

---

# 2. GROUP WORKFLOW

```text
User Creates Group
        ↓
Becomes Group Admin
        ↓
Other Users Search Group
        ↓
Send Join Request
        ↓
Admin Receives Request
        ↓
Approve / Reject
        ↓
Member Added To Group
```

---

# 3. GROUP STRUCTURE

Each Group Contains:

```text
Group
 ├── Posts Tab
 ├── Rides Tab
 ├── Members Tab
 ├── Join Requests Tab (Admin)
 └── Group Settings
```

---

# 4. COMMUNITY FEED FLOW

```text
Users Upload:
- Photos
- Videos
- Ride Clips
        ↓
Posts Visible In:
- Group Feed
- Community Feed
        ↓
Posts Can Be Tagged To Specific Ride
```

---

# 5. RIDE CREATION FLOW

```text
Admin Creates Ride
        ↓
Enter:
- Start Point
- End Point
- Ride Duration
- Stay Locations
- Stops/Restaurants
- Planned Expense
- Max Participants
        ↓
Ride Saved As:
UPCOMING RIDE
        ↓
Members Receive Notification
```

---

# 6. RIDE PARTICIPATION FLOW

```text
User Opens Upcoming Ride
        ↓
Can View:
- Ride Details
- Participants
- Expenses
- Locations
        ↓
Click Join Ride
        ↓
Payment via Google Pay
        ↓
User Added To Participants List
```

---

# 7. ATTENDANCE FLOW

## Attendance Opens Before Ride Start

```text
Configurable Time Trigger
(Default: 1 Hour Before Ride)
        ↓
Attendance Button Enabled
        ↓
Participants Mark Attendance
        ↓
Admin Can View Attendance List
```

---

# 8. RIDE STATE MANAGEMENT

## Ride Lifecycle

```text
DRAFT
   ↓
UPCOMING
   ↓
ONGOING
   ↓
COMPLETED
```

Optional:

```text
CANCELLED
```

---

# 9. START RIDE FLOW

```text
Admin Opens Upcoming Ride
        ↓
Clicks "Start Ride"
        ↓
Ride State Changes:
UPCOMING → ONGOING
        ↓
Ride Moves To:
Ongoing Ride Section
```

---

# 10. ONGOING RIDE FLOW

## Ongoing Ride Features

```text
Ride Navigation
        ↓
Display:
- Start Point
- Stops
- End Point
- Stay Locations
- Important Notes
```

---

# 11. EMERGENCY / STOP ALERT FLOW

## During Ongoing Ride

```text
Member Clicks Emergency/Stop Button
        ↓
Alert Sent To Ride Participants
        ↓
Notification:
"Member Stopping For Fuel/Refreshment"
        ↓
Group Can Stop & Wait
```

Future Expansion:
- live location
- map tracking
- SOS alert
- breakdown assistance

---

# RHINO — MODULE ARCHITECTURE

# Backend Modules

```text
auth
users
groups
rides
posts
notifications
payments
attendance
media
```

---

# DATABASE RELATIONSHIPS (HIGH LEVEL)

```text
User
 ├── Groups
 ├── Posts
 ├── Ride Participation
 └── Emergency Info

Group
 ├── Members
 ├── Posts
 ├── Rides
 └── Requests

Ride
 ├── Participants
 ├── Stops
 ├── Tagged Posts
 ├── Attendance
 └── Status

Post
 ├── User
 ├── Group
 └── Optional Ride Tag
```

---

# RECOMMENDED MVP (VERSION 1)

# Build ONLY These First

## Authentication
- Signup/Login
- JWT

## User Profiles
- Bike Info
- Emergency Details

## Groups
- Create Group
- Join Requests
- Admin Approval

## Posts
- Upload Photos/Videos

## Rides
- Create Ride
- Join Ride
- Participant Limit

## Notifications
- Ride Alerts

---

# VERSION 2 FEATURES

After MVP stabilizes:

- Attendance system
- Ongoing ride mode
- Emergency stop alerts
- Payment tracking
- Ride tagging improvements
- Better notifications

---

# VERSION 3 FEATURES

Later:

- Live maps
- Real-time tracking
- WebSockets
- Route analytics
- AI suggestions
- Weather integration
- Smart ride planning

---

# RECOMMENDED TECH STACK

## Frontend
- Flutter

## Backend
- Java
- Spring Boot

## Database
- PostgreSQL

## Auth
- JWT

## Notifications
- Firebase Cloud Messaging

## Media
- Cloudinary initially

## Deployment
- Docker later

---

# IMPORTANT ARCHITECTURE DECISION

# Start With:
# Modular Monolith

NOT microservices initially.

Reason:
- faster development
- easier debugging
- easier learning
- less deployment complexity
- cleaner iteration

Microservices can come later if:
- scaling becomes necessary
- modules become too large
- traffic grows significantly

---

# Suggested Development Order

## Phase 1
- Backend setup
- Authentication
- Database schema
- User profiles

## Phase 2
- Groups
- Group requests
- Member management

## Phase 3
- Ride system
- Ride creation
- Ride joining

## Phase 4
- Posts/media
- Community feed

## Phase 5
- Notifications
- Attendance
- Ride states

## Phase 6
- Ongoing ride features
- Emergency coordination

This order will prevent architectural chaos later.
