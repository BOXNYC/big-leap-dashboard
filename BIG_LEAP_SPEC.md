# Big Leap - Healthcare Provider Portal

## Structural Overview

- Users **with Authentication**
  - info (Full name, profile image, email, etc.)
  - preferences (Notifications, perfered method of contact, etc.)
  - practices (Practices where a member)
  - role
    - admin (Site Administrator, Full Access)
    - agent (Big Leap Staff, CRUD Practices & Users)
    - user (Practice Staff, CRUD to own Practices)
    - patient (Patient of Practices, CRUD Insurance)
  - insurance (Patients only. Insurance info, retaining change update history)
- Practice
  - info (Name, location, etc.)
  - 
