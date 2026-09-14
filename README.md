# reitschule - Suite of tools for riding clubs and schools

## Description

Riding schools often have quota-based contracts with their students. This means the students pay a subscription fee, and
can participate in riding lesson units up to an agreed-upon quota. At the same time, trainers are often freelancers and
have very individual contracts with the schools. Keeping track which student took how many lessons, with which trainer,
is often complicated and error-prone. We want to fix that by having a simple app where trainers can input the
information about lessons that took place (with which students, with which horses) - ideally directly on their phones
while still on-site. Plus another app that allows the school's admins to gather all information, generate reports
(per-trainer, per-student, per-horse, etc.), and maintain master data.

Note that this system is **not** intended as a compensation calculator for trainers! This system only tracks the number
of lessons a trainer provided. Compensation, contractual details, and so on, must be tracked elsewhere.

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

## PII, data minimization, data retention

Only the absolutely minimal amount of personally identifiable information (PII) is collected and stored, where it's
needed for the school's business processes. This includes admin's and trainer's names and e-mail addresses. For
students, only the name is stored. Every other information the school might have on any person, be it admin, trainer, or
student, is not stored in this system, but needs to be maintained elsewhere!

Trainers can see all data on active students and active horses. No finer-grained access control mechanism needed.

By default, master-data records are not deleted. When an admin, trainer or student stops interacting with the school,
their information remains as it's part of the school's business processes. The respective data object is marked as
"inactive", though. When personal data must no longer be retained, identifying attributes are anonymized while
non-personal historical records remain intact. This means database entries do not get deleted, but updated so that no
PII is retained. For example, a username gets anonymized to something like "<<deleted user>>", maybe with a sequence
number. This way, the `admin-app` can still show historical data, albeit without the deleted PII.

## Authentication

We're using external login providers through OAuth 2.0 and/or OpenID Connect. Each frontend application makes an initial
request to a dedicated endpoint of the `api-service` to check if the user is authenticated, before rendering any part of
the actual application and/or providing access to any information whatsoever. Non-authenticated users only see a "login"
page that provides several "social login" buttons like "Sign in with Google", "Sign in with Microsoft", and so on.

Admins must maintain the list of known admins, as well as the list of known trainers, each with their verified e-mail
address, which will be returned by the external login providers. **Only** known, active admins may successfully log in
to the `admin-app`, and **only** known, active trainers may successfully log in to the `trainer-app`. If another person
attempts to log in to any app that they're not whitelisted for, they will see an appropriate error message in the
frontend (even if the login _technically_ was okay from the login provider's view). No session is created in this case.

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

Docker Compose is used to spin up the `api-service` on a webserver with PHP capabilities similar to the production
environment + a database. Both frontends are running in their Vite dev server and are using its proxy capabilities to
utilize the endpoints of the running `api-service`.

- **http://localhost:8123/api** The local `api-service` instance, exposed from docker
- **http://localhost:8234/admin** The local `admin-app`
- **http://localhost:8345/trainer** The local `trainer-app`
 
# Specifications

- [v1 (rough draft)](docs/specs/v1.md)
- [v2 (based on first review comments)](docs/specs/v2.md)
- [v3 (latest increment)](docs/specs/v3.md)
