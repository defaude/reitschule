# reitschule - Suite of tools for riding clubs and schools

## Description

Riding schools often have quota-based contracts with their students. This means the students pay a monthly fee, and can
take two riding lesson units per week, for example. At the same time, trainers are often freelancers and have very
individual contracts with the schools. Keeping track which student took how many lessons, with which trainer, is often
complicated and error-prone. We want to fix that by having a simple app where trainers can input the information about
amounts of lessons that took place (with which students, with which horses) - ideally directly on their phones while
still on-site. Plus another app that allows the school's admins to gather all information, generate reports
(per-trainer, per-student, per-horse, etc.), and maintain master data.

Master data includes:
= Which admin users exist? (user management)
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
- **Horse**: Information on each horse that's being used for riding `lessons` with `students`.
- **Lesson**: When a `student` participates in a riding class with one of the `trainers` on one of the `horses`, it's a
  lesson, which is deducted from the `student's` `quota`. One `quota` unit of such a lesson is usually 45 minutes. If
  the `student` takes a double lesson of 90 minutes, for example, two units are taken from their `quota`.
- **Admin**: A user that is allowed to log in to the `admin-app`. The `api` allows them CRUD basically all data. They
  belong to the `school` and are trusted.
- **Trainer**: A user that can only log in to the `trainer-app`. The `api` only allows them to access the information
  they need to work with their `lessons` / `horses` / `students`, but nothing else. They are affiliated with the
  `school`, but are generally considered external contractors.
- **Student**: Has a `contract` with a specific monthly `quota`. Takes `lessons` with `trainers` and `horses`.
- **Contract**: Agreement between a `student` and the `school`. It specifies the default `quota` that a `student` is
  given each month. A contract always has a start date. If it's the current, active contract, there is no end date. A
  contract can be terminated (deactivated), at which point the end date is stored.
- **Quota**: Is basically the "bank account" of `lesson` units a `student` can take. If the `student` has a current,
  active `contract`, their quota is increased automatically by the amount specified in the `contract`. Each `lesson`
  unit the `student` participates in (i.e. the `trainer` entered it into the `trainer-app`) reduces the quota. Unused
  units in a `student's` quota don't automatically vanish, but carry over and accumulate. This way, a `student` can
  utilize their quota even in case of vacations, sickness, spontaneous cancellations, etc.

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
 