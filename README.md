# reitschule - Suite of tools for riding clubs and schools

## Description

Riding schools often have quota-based contracts with their students. This means the students pay a subscription fee, and
can participate in riding lesson units up to an agreed-upon quota. At the same time, trainers are often freelancers and
have very individual contracts with the schools. Keeping track which student took how many lessons, with which trainer,
is often complicated and error-prone. We want to fix that by having a simple app where trainers can input the
information about amounts of lessons that took place (with which students, with which horses) - ideally directly on
their phones while still on-site. Plus another app that allows the school's admins to gather all information, generate
reports (per-trainer, per-student, per-horse, etc.), and maintain master data.

Master data includes:
- Which admin users exist? (user management)
- Which trainers are active at the school? (user management)
- Which students are active customers at the school - and what quota is available to each one?
- Which school horses are available?

## Sub-projects

- **api**: Defines the backend API endpoints and data structures - both for the `admin-app` and the `trainer-app`.
- **api-service**: A PHP-based implementation incl. a database connection. Implements the endpoints defined in `api`.
- **admin-app**: Back-office frontend application where admins can maintain all master data, but also create monthly
  reports. It uses the endpoints provided by the `api-service`, following the spec in `api`.
- **trainer-app**: Dedicated mobile-friendly frontend to help trainers submit information about riding lessons that
  took place. It uses the endpoints provided by the `api-service`, following the spec in `api`.

## Domain entities

- **School**: The riding school.
- **Horse**: Information on each horse that's being used for riding `lessons` with `students`. A horse can either be
  "active" or "inactive". Only active horses show up as options in the `trainer-app` for new `participation` entries.
- **Lesson**: Is created by a `trainer` (or an `admin` if the trainer misses to input their data on time). It contains
  the date and timespan when the `trainer` held a lesson for one or more `students`. `Trainers` can only see and 
  manipulate their onw lessons and `participation` entries - and only before the month has been "closed" by an `admin`.
- **Participation**: A `trainer` (or `admin`) must input each `student` that participated in a `lesson` with which horse
  and for how long (i.e. how many `quota` units the student used). For example: The trainer creates a `lesson` that took
  90 minutes. `Students` Anna, Bob and Charlotte participated overall. Anna was there for the full 90 minutes, Bob only
  for the second half and Charlotte for the first half. This means Anna's `quota` is reduced by 2, while Bob's and
  Charlotte's is reduced by 1. A `student's` participation in a `lesson` is always tied to one of the `horses`. A
  `student` can only have one participation entry belonging to a `lesson`.
- **Admin**: A user that is allowed to log in to the `admin-app`. The `api` allows them CRUD basically all data. They
  belong to the `school` and are trusted. An admin user can be made inactive, even without deleting the entry. Inactive
  admins can not log in to the `admin-app` and not access or modify any data in the system.
- **Trainer**: A user that can only log in to the `trainer-app`. The `api` only allows them to access the information
  they need to work with their `lessons` / `horses` / `students`, but nothing else. They are affiliated with the
  `school`, but are generally considered external contractors. Just like admins, trainers can be made inactive and lose
  all access to the `trainer-app` and any data if that's the case.
- **Student**: Has a `contract` with a specific monthly `quota`. Participates in `lessons` with `trainers` and `horses`.
  A student can either be "active" or "inactive". Only active students show up in the `trainer-app` as possible
  candidates when a `participation` entry is created.
- **Contract**: Agreement between a `student` and the `school`. It specifies the default `quota` that a `student` is
  given each month. A contract always has a start date. If it's the current, active contract, there is no end date. A
  contract can be terminated (deactivated), at which point the end date is stored. A `student` can only have one active
  contract at any given time.
- **Quota**: Is basically the "bank account" of `participation` units a `student` can take. If the `student` has a
  current, active `contract`, their quota is increased automatically by the amount specified in the `contract`. Unused
  units in a `student's` quota don't automatically vanish, but carry over and accumulate. This way, a `student` can
  utilize their quota even in case of vacations, sickness, or other spontaneous cancellations. One quota unit
- corresponds to 45 minutes. Only full quota units are supported.
- **Transaction**: Each change to a `student's` quota is stored as a transaction log entry. Each entry contains:
  - Timestamp: when the transaction was added to the system.
  - Reference: ID of the contract or ID of the lesson this transaction is automatically based on. Empty for "manual"
    transactions.
  - Type:
    - "contract" for automatic `contract`-based top-up
    - "lesson" when the `student` used one or more units of their quota
    - "manual" correction entry entered by an `admin`.
  - Unit delta: A positive or negative integer representing the amount of units added to or removed from the `quota`.
  - Issuer: Empty for all but "manual" entries. Populated with the `admin's` username that created the correction entry.
  - Description: Empty for all but "manual" entries. Free-text field the `admin` can fill when creating a correction.

## Quota transactions

If a student has an active contract at the beginning of a month, a "contract" transaction is created that increases the
student's quota by the amount that is defined in the contract. Such a "contract" transaction is only created once per
contract per month.

If a new contract starts on the 1st day of the month, the automatic transaction to top up the student's quote is added.

If a contract starts at a later day in the month, no automatic transaction is added in that month. In this case, an
admin needs to create an appropriate transaction for the onboarding of the student manually. When a contract ends, the
quota stays unchanged. The quota __can__ become negative, but the `admin-app` will show a warning message banner if this
is the case (so that admins can take action if needed).

Each lesson a student participates in a lesson (i.e. a trainer or an admin adds a participation entry), results in a
"lesson" transaction with the amount of units from the participation entry. It reduces the student's quota accordingly.
If the participation entry gets modified, the transaction is updated, as well. No additional timestamp for the change
needs to be recorded. If the participation entry is removed, the transaction is removed as well.

## Closing & Report generation

After the end of a month, an admin can close the month's data regarding lessons and participation. After this moment,
the month's data becomes immutable for trainers. An admin can add, edit, or remove lesson or participation entries if
the trainer's data was incomplete or incorrect, even if the month has already been closed.

A report is not a persisted document, but a manually-triggered data aggregation. Reports are displayed in the
`admin-app` directly in a table, with frontend-only filtering and sorting for usability. No export or download
functionality for exports is needed at the moment.

Some examples of reports are (but there might be other use-cases in the future):
- list of all lessons a specific trainer gave (in a month)
- list of all participation units a specific horse was used for (in a month)
- list of all participation units a specific student used (in a month)
- list of all quota transactions of a specific student's quota (over a longer period, maybe even the whole contract
  duration)

## PII, data minimization, data retention

Only the absolutely minimal amount of personally identifiable information (PII) is collected and stored, where it's
needed for the school's business processes. This includes admin's and trainer's names and e-mail addresses. For
students, only the name is stored. Every other information the school might have on any person, be it admin, trainer, or
student, is not stored in this system, but needs to be maintained elsewhere!

Trainers can see all data on active students and active horses. No finer-grained access control mechanism needed.

By default, no information is deleted. When an admin, trainer or student stops interacting with the school, their
information remains as it's part of the school's business processes. The respective data object is marked as "inactive",
though. When personal data must no longer be retained, identifying attributes are anonymized while non-personal
historical records remain intact. This means database entries do not get deleted, but updated so that no PII is
retained. For example, a username gets anonymized to something like "<<deleted user>>", maybe with a sequence number.
This way, the `admin-app` can still show historical data, albeit without the deleted PII.

## Authentication

We're using external login providers through OAuth 2.0 and/or OpenID Connect. Each frontend application makes an initial
request to a dedicated endpoint of the `api-service` to check if the user is authenticated, before rendering any part of
the actual application and/or providing access to any information whatsoever. Non-authenticated users only see a "login"
page that provides several "social login" buttons like "Sign in with Google", "Sign in with Microsoft", and so on.

The `api-service` provides the return URLs for the login flows of  each of the providers, and manages session cookies
with appropriate lifetimes. This allows the frontends to be almost "oblivious" of the whole login flow, as we're using
the same domain + cookies for all apps and the API.

## Operations

To minimize costs, all applications are running on a simple web-hoster that provides PHP, relational databases (e.g.
MariaDB). This means the `api-service` must be a PHP app that only provides the endpoints (most likely REST-style). Both
frontends (`admin-app` and `trainer-app`) are built as SPAs, which produce static content (HTML, CSS, JS, images)
directly from the webserver.

- **https://some.custom.domain.de/admin** serves the `admin-app` (static content)
- **https://some.custom.domain.de/trainer** serves the `trainer-app` (static content)
- **https://some.custom.domain.de/api** serves the `api-service` (PHP)

## Local operation / development

Docker compose is used to spin up the `api-service` on a webserver with PHP capabilities similar to the production
environment + a database. Both frontends are running in their Vite dev server and are using its proxy capabilities to
utilize the endpoints of the running `api-service`.

- **http://localhost:8123/api** The local `api-service` instance, exposed from docker
- **http://localhost:8234/admin** The local `admin-app`
- **http://localhost:8345/trainer** The local `trainer-app`
 