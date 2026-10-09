# FinAB --- Financial Advisor and Budgeting Web Application

> **Project status:** In development\
> **Documentation snapshot:** 26 September 2026\
> **Previous project names:** FABW / FinPlan\
> **Current project name:** FinAB

FinAB is a Java-based financial planning and budgeting web application.
It helps users organize their financial information, understand their
monthly financial position, explore investment options, set financial
goals, and view illustrative long-term projections.

Version 1 is designed around information entered by the user. Continuous
transaction-level expense tracking and automatic, agentic revision of
financial plans are reserved for a possible future version.

------------------------------------------------------------------------

## Table of Contents

-   [Project Goals](#project-goals)
-   [Features and Scope](#features-and-scope)
-   [Technology Stack](#technology-stack)
-   [Architecture](#architecture)
-   [Database Design](#database-design)
-   [Investment Data and Calculation
    Rules](#investment-data-and-calculation-rules)
-   [API Notes](#api-notes)
-   [Current Implementation Status](#current-implementation-status)
-   [Known Issues and Next Actions](#known-issues-and-next-actions)
-   [Local Development Environment](#local-development-environment)
-   [Setup and Run Notes](#setup-and-run-notes)
-   [Testing](#testing)
-   [Suggested Agile Workflow](#suggested-agile-workflow)
-   [Security and Financial
    Disclaimer](#security-and-financial-disclaimer)
-   [Version 1 Scope Boundaries](#version-1-scope-boundaries)

------------------------------------------------------------------------

## Project Goals

FinAB aims to:

1.  Collect a user's relevant profile and financial information in a
    streamlined assessment flow.
2.  Summarize income, expenses, monthly surplus, emergency savings, and
    potential investment capacity.
3.  Provide general investment options and recommendations based on
    deterministic Java business rules and stored assumptions/rates.
4.  Support multiple financial goals and estimate the monthly investment
    needed to work toward each goal.
5.  Present illustrative future-value projections across selected
    investment horizons.
6.  Provide a dashboard where users can view their financial summary and
    view or edit profile information.

The application is intended for financial planning education and
organization. It does not guarantee investment returns or replace a
qualified financial adviser.

## Features and Scope

### In Version 1

-   User registration and authentication.
-   Personal profile management.
-   A consolidated financial assessment form for related information.
-   Income and expense inputs for planning.
-   Financial surplus and investment-capacity calculations.
-   Optional current emergency-savings information and an emergency-fund
    target.
-   Multiple financial goals, each with a target amount, target date,
    current amount, and priority where supported.
-   Investment options:
    -   Equity exposure
    -   Mutual funds / SIP
    -   Fixed Deposit (FD)
    -   Recurring Deposit (RD)
    -   Public Provident Fund (PPF)
-   Investment recommendations and illustrative projections.
-   Dashboard summaries and profile editing.
-   API and integration testing.

### Future scope, not part of the current Version 1

-   Continuous transaction-level monthly expense tracking.
-   Automatic monitoring of real spending.
-   Agentic AI that dynamically revises financial plans.
-   Python, RAG, LLM-based planning, or other AI features. These remain
    separate from the Java application for now.

## Technology Stack

  -----------------------------------------------------------------------
  Layer                               Technology
  ----------------------------------- -----------------------------------
  Frontend                            React

  Backend                             Java, Spring Boot

  API                                 REST

  Persistence                         Spring Data JPA / Hibernate

  Database                            PostgreSQL

  Security                            Spring Security, BCrypt password
                                      hashing, JWT authentication
                                      components

  Application server                  External Apache Tomcat; deploy as a
                                      WAR

  Build                               Maven

  IDE                                 Eclipse IDE for Enterprise Java and
                                      Web Developers

  API testing                         Postman

  Database administration             pgAdmin

  Version control                     Git / GitHub

  Frontend runtime                    Node.js and npm
  -----------------------------------------------------------------------

### Environment versions recorded most recently

-   JDK: `26.0.2.1`
-   Apache Tomcat: `11.0.26`
-   Node.js: `v24.21.0`
-   PostgreSQL: `18.6`
-   Git: `2.55.0.windows.5`
-   Eclipse: `2025-09`
-   Maven: `3.9.16`

The working design document also references Spring Boot `4.1.1` and Java
21. Verify the actual `pom.xml`, JDK compatibility, and runtime
configuration in the current repository before changing versions.

## Architecture

The application uses a layered backend architecture:

``` text
React Frontend
      |
      | HTTP / JSON
      v
REST Controllers
      |
      v
Service Layer
(business rules and calculations)
      |
      v
Repository Layer
(Spring Data JPA)
      |
      v
JPA / Hibernate
      |
      v
PostgreSQL
```

Authentication and authorization are handled through Spring Security.
Passwords are hashed with BCrypt; plaintext passwords must never be
stored. JWT authentication has been added to the backend and has been
observed validating successfully in API logs.

The deployment preference is an **external Apache Tomcat server**, with
the Spring Boot application packaged as a WAR. Do not switch the project
to embedded-Tomcat deployment.

### Backend package structure

The initial package structure is under `com.fabw` (the Java package name
may remain unchanged even though the product is now called FinAB):

``` text
com.fabw
├── FabwApplication.java
├── ServletInitializer.java
├── TestController.java
├── config/
│   ├── SecurityConfig.java
│   └── PasswordConfig.java
├── controller/
├── dto/
├── entity/
├── repository/
├── service/
└── exception/
    └── GlobalExceptionHandler.java
```

The broader module design includes controllers, DTOs, entities,
repositories, services, configuration, exception handling, and utilities
where required.

## Database Design

The database name recorded in the setup notes is `fabw_db`. The project
has several application tables and rate/assumption tables. Confirm the
current schema in PostgreSQL before applying migrations; names and
columns in older design documents may not exactly match the live
database.

### Main application tables

  -----------------------------------------------------------------------
  Table                               Purpose
  ----------------------------------- -----------------------------------
  `users`                             Account identity, email, password
                                      hash, and timestamps. The intended
                                      name column is `full_name`, not
                                      `name`.

  `user_profiles`                     Profile information such as age,
                                      employment/occupation, city,
                                      marital status, dependents, and
                                      risk profile where present. The
                                      dashboard should allow profile
                                      viewing and editing.

  `income`                            Primary in-hand salary and optional
                                      other-income information, including
                                      frequency fields where implemented.

  `expense_categories`                Reference list of expense
                                      categories.

  `expenses`                          User-entered expense amounts
                                      associated with categories for
                                      planning; not continuous
                                      transaction tracking in Version 1.

  `emergency_funds`                   Current emergency savings, target
                                      coverage/amount, and related
                                      timestamps. Current savings may be
                                      nullable; a missing value should be
                                      handled safely in calculations.

  `financial_goals`                   Multiple goals per user, including
                                      goal name, target amount, current
                                      amount, target date, priority, and
                                      timestamps.

  `investment_types`                  General investment option metadata,
                                      such as risk level and assumptions
                                      where applicable.

  `investment_recommendations`        Recommendation amounts, allocation
                                      percentages, reasoning, and links
                                      to users/investment types where
                                      implemented.

  `investment_projections`            Investment amount/frequency,
                                      assumed return, period, projected
                                      value, and related fields where
                                      implemented.

  `bank_investment_rates`             Bank-specific investment rates,
                                      particularly the source for FD/RD
                                      rate data.

  `investment_return_assumptions`     Assumed returns for market-linked
                                      investment scenarios such as equity
                                      exposure and mutual fund/SIP
                                      projections.
  -----------------------------------------------------------------------

### Important schema decisions

-   The intended users-table column is `full_name`.
-   A separate `salary_details` table is not part of the agreed design.
-   Avoid persisting CTC or intermediate salary-calculator inputs as
    permanent profile data unless the schema and requirements are
    explicitly revised.
-   The profile/dashboard must support viewing and editing user profile
    details.
-   Multiple financial goals are supported.
-   The miscellaneous expense category must be retained.
-   Version 1 uses category-level expense estimates rather than a
    continuous expense-tracking workflow.
-   The financial assessment should be a consolidated flow rather than
    many disconnected forms.

### Important investment-rate distinction

-   **FD and RD:** Use `bank_investment_rates` for bank-specific rates.
    Do not calculate FD/RD projections from generic entries in
    `investment_return_assumptions` when the bank-rate table is the
    intended source.
-   **Equity exposure and mutual funds/SIP:** Use appropriate records
    from `investment_return_assumptions` and clearly label projections
    as estimates.
-   **PPF:** Use an appropriate configured rate/source and clearly
    identify the effective date and assumptions.
-   Rate records should include source URLs and effective dates where
    the schema supports them. Refresh rates when authoritative sources
    change.

The schema has evolved during implementation. The table descriptions
above capture the known design and development decisions, not a
substitute for inspecting the live database.

## Investment Data and Calculation Rules

FinAB's intended calculations include:

-   Normalize applicable income and expense values to a common monthly
    basis before calculating monthly surplus.
-   Calculate investable capacity only after considering entered
    expenses and the applicable emergency-fund requirement.
-   Do not assume every rupee of surplus should be invested; preserve
    room for miscellaneous/lifestyle spending and reasonable liquidity.
-   Recommend amounts that fit the user's financial capacity.
-   For lower monthly surpluses, avoid suggesting impractical lump-sum
    FD amounts; consider appropriate recurring or smaller-contribution
    options where the product rules and user circumstances allow.
-   Support multiple financial goals and estimate the required monthly
    investment from target amount, current amount, time horizon, and
    applicable assumed return.
-   Show the assumptions behind future projections. Market-linked
    returns are uncertain and not guaranteed.

A previously tested goal example was a **Car** goal with a target amount
of **₹8,00,000** and target date **2034-10-01**. A projection response
was recorded with an assumed annual return of `10%`, a required monthly
investment of `₹5,472.66`, and an eight-year horizon. Treat these as a
historical test example, not a universal recommendation or a guaranteed
result.

## API Notes

The initial high-level API design includes the following resource areas.
Treat these as design-level routes until each one is confirmed against
the current controllers and frontend API client.

  Resource area                Planned route
  ---------------------------- -----------------------------------------
  Authentication               `/api/auth/register`, `/api/auth/login`
  Profile                      `/api/profile`
  Income                       `/api/income`
  Expenses                     `/api/expenses`
  Financial goals              `/api/goals`
  Emergency fund               `/api/emergency-fund`
  Investment recommendations   `/api/investments/recommendations`
  Investment projections       `/api/investments/projections`
  Dashboard                    `/api/dashboard`

### Routes observed during implementation

The signup endpoint was tested as:

``` http
POST /fabw/users/signup
```

A successful signup returned `201 Created`, and the new user was
verified in PostgreSQL. The response contained `userId`, `fullName`, and
`email`; it did not expose the password hash.

Other implementation routes observed in frontend/backend work include:

``` text
/users/signup
/financial-goals
/expenses
/emergency-funds
/income
```

The frontend has used calls such as `POST /financial-goals`,
`POST /expenses`, `POST /emergency-funds`, and `POST /income`. Exact
context-path and API prefixes depend on the current deployment
configuration and API client. Check the controller mappings and frontend
service configuration before using these as final endpoint
documentation.

## Current Implementation Status

### Confirmed implemented or tested in earlier work

-   Spring Boot application setup.
-   Maven project configuration and WAR deployment configuration.
-   External Tomcat configuration and successful startup during setup.
-   PostgreSQL datasource and JPA/Hibernate connectivity.
-   `User` entity, repository, service, controller, signup DTO, and
    response DTO.
-   Spring Security configuration and BCrypt password hashing.
-   Signup validation.
-   Signup endpoint successfully returned `201 Created`.
-   User insertion verified in PostgreSQL.
-   JWT validation success was observed in later API logs.
-   Financial-goal creation and projection logic were exercised.
-   React financial-assessment and dashboard code exists in the project
    work.
-   The assessment form has included expense categories, additional
    income sources, emergency-fund information, and multiple goals.
-   The dashboard has included profile details, projection information,
    goal information, and financial warnings.

"Implemented" does not mean every path is fully tested or free of
defects. Re-test the current code and database before release.

## Known Issues and Next Actions

The latest reported debugging focus was the dashboard showing **monthly
expenses as `0`** while other data appeared, and the monthly in-hand
salary calculation not reflecting secondary-income information as
expected. These should be treated as open verification tasks until
retested and resolved.

### Recommended order of work

1.  **Verify financial assessment persistence**
    -   Submit non-zero expense values.
    -   Confirm requests reach the expected backend endpoints.
    -   Inspect response codes and request payloads.
    -   Verify rows in `expenses` using the correct user ID and category
        IDs.
    -   Confirm whether repeated assessment submissions should update
        existing category rows rather than create duplicates.
2.  **Fix dashboard expense totals**
    -   Check the dashboard API response before debugging React display
        code.
    -   Confirm the backend query filters by the authenticated user.
    -   Check month/year filters, category joins, and numeric field
        mapping.
    -   Ensure the frontend uses the actual response field rather than a
        hard-coded zero or mismatched property name.
3.  **Verify income aggregation**
    -   Inspect the saved primary and secondary income records.
    -   Confirm income frequencies are normalized correctly.
    -   Trace the calculation from repository query through service and
        DTO to dashboard rendering.
    -   Define and test the expected total-income rule consistently
        across backend and frontend.
4.  **Check emergency-fund handling**
    -   Test both a provided current-savings amount and a missing/null
        value.
    -   Ensure target calculations use the intended essential-expense
        basis and coverage period.
5.  **Fix API error responses**
    -   Invalid signup data previously triggered validation but returned
        an unclear `403 Forbidden`.
    -   A centralized `GlobalExceptionHandler` was identified as the
        intended solution. Verify whether it has since been completed.
    -   Handle duplicate-email and data-integrity errors with clear
        client-safe responses.
6.  **Verify financial-goal constraints**
    -   A goal insertion previously failed on the database constraint
        `check_goal_priority`.
    -   The frontend mapped priorities to `High`, `Medium`, and `Low`;
        confirm these match the exact values permitted by the database
        constraint.
    -   Re-test goal creation, retrieval, and projections.
7.  **Verify investment projections**
    -   Confirm FD/RD use `bank_investment_rates`.
    -   Confirm market-linked assumptions come from
        `investment_return_assumptions`.
    -   Validate compounding, contribution frequency, horizon, rounding,
        and missing-rate handling.
    -   Clearly label estimated values.
8.  **Complete frontend integration**
    -   Ensure authentication tokens are attached to protected requests.
    -   Test registration/login, assessment submission, dashboard
        loading, profile editing, goals, and projections as an
        end-to-end workflow.
    -   Add clear loading, empty, success, and error states.
9.  **Run a release regression pass**
    -   Test a new user and a returning user.
    -   Verify one user's data cannot be accessed by another user.
    -   Test invalid, missing, zero, and boundary values.
    -   Commit changes to Git with clear messages.

## Local Development Environment

The following versions were recorded in the development setup:

``` text
JDK:          26.0.2.1
Apache Tomcat: 11.0.26
Node.js:      v24.21.0
PostgreSQL:   18.6
Git:          2.55.0.windows.5
Eclipse:      IDE for Enterprise Java and Web Developers 2025-09
Maven:        3.9.16
Database:     fabw_db
```

The original development machine used Windows. Environment paths and
local database credentials are machine-specific and should not be
committed to the repository.

## Setup and Run Notes

These are the general steps; exact commands depend on the current
repository layout and configuration.

### Backend

1.  Install/configure the compatible JDK and Maven.

2.  Ensure PostgreSQL is running and the `fabw_db` database exists.

3.  Configure datasource credentials in local configuration or
    environment variables.

4.  Check `pom.xml` for Java/Spring Boot versions and WAR packaging.

5.  Build the backend using the Maven wrapper if available:

    ``` bash
    ./mvnw clean package
    ```

    On Windows Command Prompt, use `mvnw.cmd clean package` if the
    wrapper exists. Otherwise use an installed Maven command.

6.  Deploy the generated WAR to the configured external Apache Tomcat
    instance.

7.  Check application logs and verify a known endpoint.

### Frontend

1.  Open the React frontend directory.

2.  Install dependencies:

    ``` bash
    npm install
    ```

3.  Configure the frontend API base URL to match the actual backend
    context path and controller mappings.

4.  Start the development server:

    ``` bash
    npm run dev
    ```

    Use `npm start` instead if that is the script defined by the
    project's `package.json`.

5.  Test authenticated requests and confirm the JWT token is passed
    where required.

### Configuration safety

-   Never commit real database passwords, JWT secrets, API keys, or
    other credentials.
-   Use local environment variables or an ignored local configuration
    file.
-   Do not copy the placeholder values from documentation as working
    credentials.
-   Verify `.gitignore` covers local secrets, build output,
    dependencies, and environment files.

## Testing

Use Postman to test backend APIs independently of React, then test the
complete user workflow through the frontend.

Suggested checks:

-   Signup with valid and invalid input.
-   Duplicate email handling.
-   Login and JWT authentication.
-   Protected route behavior with missing, invalid, and valid tokens.
-   Profile retrieval and update.
-   Income and expense persistence.
-   Monthly income, expense, surplus, and investment-capacity
    calculations.
-   Emergency-fund calculations with and without current savings.
-   Creation of multiple financial goals and retrieval by the correct
    user.
-   Projection results for multiple horizons.
-   Correct investment-rate source for each investment type.
-   Dashboard totals matching saved database data.
-   Cross-user authorization and data isolation.

Do not mark a feature complete solely because its endpoint returns a
successful HTTP status; verify saved data and the business result.

## Suggested Agile Workflow

Use short, fixed sprints (one or two weeks are practical for a solo
student project). Maintain a backlog with each story's priority,
acceptance criteria, estimate, status, and sprint.

Suggested sequence based on the current known work:

1.  **Sprint A --- Data integrity and calculation fixes:** resolve
    expense totals, income aggregation, and dashboard response mapping.
2.  **Sprint B --- API reliability and security:** consistent validation
    errors, duplicate handling, JWT/authorization tests, and goal
    priority constraints.
3.  **Sprint C --- Investment engine validation:** verify bank rates,
    assumptions, recommendation constraints, and projections.
4.  **Sprint D --- Frontend completion:** consolidated assessment,
    dashboard, profile viewing/editing, goal management, and API error
    states.
5.  **Sprint E --- End-to-end testing and release:** regression tests,
    documentation updates, deployment verification, and demo
    preparation.

These are proposed work packages, not a claim that each feature is
currently unfinished. Review the live repository before assigning sprint
tasks.

A story is done when its acceptance criteria pass, integration is
verified, relevant tests pass, error cases are handled, and the changes
are committed.

## Security and Financial Disclaimer

-   Store password hashes, never plaintext passwords.
-   Protect user-specific routes and ensure authenticated users can only
    access their own data.
-   Validate all client input on the backend; frontend validation alone
    is insufficient.
-   Do not expose stack traces, credentials, tokens, or internal
    database details in API errors.
-   Keep financial assumptions and data sources auditable.
-   All projections are illustrative estimates. Investment values can
    rise or fall, and actual returns may differ materially. Users should
    review product terms, risk, fees, tax treatment, and eligibility
    independently.

    
© 2026 Urvi Narvekar. All Rights Reserved.
FinAB — Financial Advisor and Budgeting Web Application.

