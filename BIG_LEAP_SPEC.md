# Big Leap - Healthcare Provider Dashboard

[Thought Exercise Overview](https://reminiscent-aries-c5f.notion.site/Full-Stack-Engineer-Thought-Exercise-29be4ce57d97808db4b7c3bc6f2834eb)

## Structural Overview

- Users **with Authentication**
  - **profile** *Full name, profile image, etc.*
  - **account** *email, phone, etc.*
  - **preferences** *Notifications, perfered method of contact, etc.*
  - **practices** *Practices where a member*
  - **role**
    - **admin** *Site Administrator, Full Access*
    - **agent** *Big Leap Staff, CRUD Practices & Users*
    - **user** *Practice Staff, CRUD to own Practices*
    - **patient** *Patient of Practices, CRUD Insurance*
  - **dob** *[Patients only] Date of Birth*
  - **insurance** *[Patients only] Insurance info, retaining change update history*
  - **eligible** *If a patient is eligible for treatment from Big Leap Health*
- **Practice**
  - **info** *Name, location, etc.*

## Actions

- a new patient request from the practice
- patient insurance update from the practice
- a patient is eligible for treatment from Big Leap Health
- incorrect patient details such as Date of Birth
