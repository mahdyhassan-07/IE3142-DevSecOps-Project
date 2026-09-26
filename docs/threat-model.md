# NodeGoat Threat Model

## 1. Assets

The following assets were identified based on the functionality and data
handled by the NodeGoat application.

| ID | Asset | Description | Security Importance |
|----|-------|-------------|---------------------|
| A1 | User Credentials | Usernames and passwords used to authenticate users. | Compromise could allow unauthorized access to user accounts. |
| A2 | Authentication Sessions | Session identifiers and authenticated session state used to maintain logged-in users. | Compromise could allow an attacker to impersonate an authenticated user. |
| A3 | User Profile Information | Personal information stored in user profiles, including name, date of birth, address, SSN and banking-related information. | Unauthorized disclosure could expose sensitive personal and financial information. |
| A4 | Retirement and Contribution Data | Retirement savings information and payroll contribution percentages associated with users. | Unauthorized modification or disclosure could affect the integrity and confidentiality of financial information. |
| A5 | Asset Allocation Data | Information about a user's stock, fund and bond allocations. | Unauthorized access could disclose private financial information belonging to other users. |
| A6 | MongoDB Application Data | Application records stored by the MongoDB database, including user and financial-related application data. | Database compromise could affect the confidentiality, integrity  and availability of application information. |
| A7 | Administrator Privileges | Elevated application permissions available to administrator accounts. | Unauthorized access could provide capabilities beyond those available to normal users. |
| A8 | Application Configuration and Secrets | Security-sensitive application configuration such as session secrets, cryptographic configuration and database connection settings. | Exposure or manipulation could weaken application security or enable further attacks. |

## 2. Entry Points

The following entry points represent interfaces through which user-controlled
data or requests enter the NodeGoat application.

### EP1 - Login Form

Accepts username and password credentials used to authenticate users.

- **Input:** Username and password
- **Relevant component:** Authentication/session handling
- **Relevant code:** `app/routes/session.js`
- **Security relevance:** Malicious or manipulated authentication requests may target user accounts or session handling.

### EP2 - Contribution Update Form

Accepts payroll contribution percentages for pre-tax, Roth, and after-tax
contributions.

- **Input:** Contribution percentage values
- **Relevant component:** Contribution processing
- **Relevant code:** `app/routes/contributions.js`
- **Security relevance:** User-controlled contribution values reach server-side processing and are relevant to server-side JavaScript injection.

### EP3 - Allocation Stock Threshold

Accepts a stock threshold value used to filter asset allocation information.

- **Input:** Stock threshold value
- **Relevant component:** Allocation filtering and MongoDB queries
- **Relevant code:** `app/routes/allocations.js` and `app/data/allocations-dao.js`
- **Security relevance:** The threshold reaches a MongoDB `$where` expression and is relevant to NoSQL injection.

### EP4 - Memo Submission

Accepts user-generated memo content.

- **Input:** Memo text
- **Relevant component:** Memo functionality
- **Relevant code:** `app/routes/memos.js`
- **Security relevance:** User-generated content may create injection or output-encoding risks if it is stored or rendered without appropriate protection.

### EP5 - Profile Update Form

Accepts personal information including name, SSN, date of birth, bank account
number, bank routing number, address, and website.

- **Input:** Personal and financial information
- **Relevant component:** User profile management
- **Relevant code:** `app/routes/profile.js`
- **Security relevance:** This entry point handles sensitive user information and therefore affects confidentiality and integrity.

### EP6 - Stock Research Form

Accepts a stock symbol supplied by the user.

- **Input:** Stock symbol
- **Relevant component:** Stock research functionality
- **Relevant code:** `app/routes/research.js`
- **Security relevance:** User-controlled input is processed by the research functionality and should be treated as untrusted.

### EP7 - Application Routes and Authenticated Requests

The browser sends HTTP requests to NodeGoat when users navigate between pages,
retrieve account information, update information, and perform authenticated
actions.

- **Input:** HTTP requests, route parameters, query parameters, form data, and session information
- **Relevant component:** Node.js/Express web application
- **Security relevance:** Requests cross the browser-to-application trust boundary and may contain attacker-controlled data.

### EP8 - External Learning Resource Link

The Learning Resources functionality causes the user's browser to navigate from
RetireEasy to an external website.

- **Input:** Navigation request to an external destination
- **Relevant component:** External link/redirect functionality
- **Security relevance:** External navigation is relevant when considering redirect and trust-boundary risks.

## 3. System Architecture and Data Flows

### 3.1 System Components

The NodeGoat RetireEasy environment consists of a user's web browser,
the NodeGoat web application, and a MongoDB database. The NodeGoat web
application and MongoDB database run as separate Docker Compose services.

#### C1 - User Web Browser

The user interacts with RetireEasy through a web browser. The browser is
outside the Docker environment and sends HTTP requests to the NodeGoat
web application.

User-controlled information entering through the browser includes login
credentials, contribution values, allocation thresholds, memo content,
profile information, and stock research input.

#### C2 - NodeGoat Web Application

The `web` service contains the NodeGoat RetireEasy application and is
built from the project's Dockerfile.

The application is implemented using Node.js and Express and is exposed
to the host through port 4000.

The web application:

- Receives and processes HTTP requests from users.
- Performs authentication and session management.
- Processes application business logic.
- Reads and updates retirement-related information.
- Communicates with the MongoDB database.

#### C3 - MongoDB Database

The `mongo` service provides the MongoDB database used by NodeGoat.

The current Docker Compose configuration uses the `mongo:4.4` image.
MongoDB listens on port 27017 within the Docker environment.

The NodeGoat application connects to the database using:

`mongodb://mongo:27017/nodegoat`

where `mongo` is the Docker Compose service name and `nodegoat` is the
database name.

The database stores application information including user records and
retirement-related data.

### 3.2 Main Data Flows

#### DF1 - Browser to NodeGoat

The user's browser sends HTTP requests to the NodeGoat web application
through port 4000.

Requests may contain authentication credentials, form data, URL
parameters, query parameters, and session information.

These requests include actions such as logging in, viewing retirement
information, updating contribution percentages, submitting memos,
updating profile information, filtering allocations, and performing
stock research.

#### DF2 - NodeGoat to Browser

The NodeGoat web application processes requests and returns HTTP responses
to the user's browser.

Responses may contain dashboard information, retirement information,
contribution information, asset allocations, profile information, and
other application results.

#### DF3 - NodeGoat to MongoDB

The NodeGoat web application communicates with the MongoDB service to
store, retrieve, and update application data.

Communication occurs between the Docker services using port 27017 and
the MongoDB connection:

`mongodb://mongo:27017/nodegoat`

#### DF4 - MongoDB to NodeGoat

MongoDB returns requested database records and query results to the
NodeGoat web application.

The application uses these results when generating responses for the
user.

#### DF5 - Browser to External Learning Resource

The Learning Resources functionality can cause the user's browser to
navigate from the RetireEasy application to an external learning-resource
website.

This flow leaves the NodeGoat application environment and reaches an
external system.

## 4. Trust Boundaries

A trust boundary represents a point where data moves between components
with different levels of trust or security responsibility. Data crossing
these boundaries must not automatically be considered trustworthy.

### TB1 - External Client to NodeGoat Web Application

The primary trust boundary exists between the user's web browser and the
NodeGoat web application.

The browser operates outside the trusted application environment and sends
HTTP requests to the NodeGoat application through port 4000. Therefore,
information received from the client must be treated as untrusted because
a user can control or manipulate requests and submitted values.

Data crossing this boundary includes:

- Authentication credentials.
- Contribution percentage values.
- Stock allocation threshold values.
- Memo content.
- Profile and personal information.
- Stock research input.
- URL and query parameters.
- Session identifiers and cookies.

**Related data flows:** DF1 (Browser to NodeGoat) and DF3 (NodeGoat to Browser).

**Security considerations:** Authentication, authorization, input validation,
output encoding, and secure session management must be enforced at this
boundary.

This boundary is directly relevant to threats such as server-side injection,
NoSQL injection, session-related attacks, and unauthorized manipulation of
application requests.

### TB2 - NodeGoat Web Application to MongoDB

A second trust boundary exists between the NodeGoat web application and the
MongoDB database because they operate as separate Docker services with
different security responsibilities.

The NodeGoat application sends database operations and application data to
MongoDB through the Docker network on port 27017. MongoDB then returns query
results and stored information to the application.

Data crossing this boundary includes:

- User account records.
- Retirement and contribution information.
- Asset allocation information.
- Profile and financial information.
- Application data retrieved through database queries.

**Related data flows:** DF2 (NodeGoat to MongoDB).

**Security considerations:** Database queries must be constructed safely,
application access to the database should follow least privilege, and
sensitive data must be protected against unauthorized access or modification.

This boundary is particularly relevant to database-related threats such as
NoSQL injection and unauthorized access to stored application data.

## 5. STRIDE Threats

### T1 - Server-Side JavaScript Injection in Contribution Update

**STRIDE Category:** Tampering

**Component:** NodeGoat web application

**Entry Point:** Contribution Update Form (EP2)

**Assets at Risk:** Contribution data, application integrity, and potentially
other server-side resources accessible to the NodeGoat process.

**Affected Functionality:** Contribution percentage update

**Source Location:** `app/routes/contributions.js`

**Threat Description:**

The contribution update functionality accepts user-controlled payroll
contribution values. The NodeGoat implementation processes these values using
JavaScript `eval()`. Because `eval()` interprets supplied input as JavaScript
rather than treating it only as contribution data, malicious input could cause
the server to execute unintended JavaScript.

This threatens the integrity of application processing and, depending on the
injected code, could also affect the confidentiality or availability of the
application.

**Proposed Mitigation:**

Remove the use of `eval()` for processing contribution values. Treat
contribution values as data rather than executable JavaScript and validate
that the submitted values meet the expected numeric format and allowed range.

For this NodeGoat functionality, numeric conversion such as `parseInt()` can
replace the unsafe `eval()` calls together with appropriate server-side input
validation.

**Control Location:** `app/routes/contributions.js`

### T2 - NoSQL Injection in Asset Allocation Filtering

**STRIDE Category:** Information Disclosure

**Component:** NodeGoat web application and MongoDB

**Entry Point:** Allocation Stock Threshold (EP3)

**Assets at Risk:** Asset allocation data and user information.

**Affected Functionality:** Asset allocation filtering by stock threshold.

**Source Location:** `app/data/allocations-dao.js`

**Threat Description:**

The asset allocation functionality accepts a user-controlled stock threshold

value. The value is inserted directly into a MongoDB `$where` expression

without first being restricted to the expected numeric value.

An attacker could manipulate the threshold input so that the resulting query

behaves differently from its intended logic. This could cause allocation

records outside the intended user's results to be returned, potentially

disclosing other users' allocation and associated user information.

Because `$where` evaluates JavaScript expressions in MongoDB, malicious input

may also affect database availability depending on the injected expression.

### T3 - Session Fixation / Weak Session Management

- **STRIDE Category:** Spoofing
- **Affected Component:** Authentication and session management
- **Entry Point:** User login
- **Affected Code:** `app/routes/session.js` - `handleLoginRequest()`
- **Asset at Risk:** Authenticated user sessions and user accounts

#### Threat Description
The application does not regenerate the session identifier after a successful
login. Instead, the existing session is assigned the authenticated user's ID.
If an attacker is able to obtain or control a session identifier before the
victim authenticates, the same session may remain valid after login, creating
a session fixation risk and potentially allowing the attacker to impersonate
the authenticated user.

#### Code Evidence
The login handler directly assigns the authenticated user's ID to the existing
session:

`req.session.userId = user._id;`

The application does not call `req.session.regenerate()` before assigning the
authenticated identity during login.

#### STRIDE Justification
This maps to **Spoofing** because successful exploitation could allow an
attacker to use another user's authenticated session and act as that user.

#### Proposed Mitigation

Regenerate the session identifier after successful authentication before

associating the authenticated user's identity with the session.

The login handler should use `req.session.regenerate()` and assign the

authenticated user's ID only after a new session has been created. This

prevents a session identifier established before authentication from

continuing as the authenticated session.

Appropriate session cookie protections and session expiration controls

should also be applied as part of secure session management.

**Control Location:** `app/routes/session.js` - `handleLoginRequest()`

### T4 - Insecure Direct Object Reference in Asset Allocations

- **STRIDE Category:** Information Disclosure
- **Affected Component:** Asset allocation functionality
- **Entry Point:** Allocation page URL containing the user ID
- **Affected Code:** `app/routes/allocations.js` - `displayAllocations()`
- **Asset at Risk:** Users' asset allocation and account information

#### Threat Description

The asset allocation functionality obtains the user ID from the URL through
`req.params` and uses this value to retrieve allocation records. The
application does not verify that the requested user ID belongs to the
currently authenticated user.

An authenticated attacker could manipulate the user ID in the request URL
and potentially retrieve another user's asset allocation information.

#### Code Evidence

The allocation handler obtains the user ID from the request parameters:

`const { userId } = req.params;`

The user-controlled ID is then passed to:

`allocationsDAO.getByUserIdAndThreshold(userId, threshold, ...)`

instead of obtaining the authenticated user's ID from `req.session`.

#### STRIDE Justification

This maps primarily to **Information Disclosure** because successful
exploitation could allow an authenticated user to access asset allocation
information belonging to another user.

#### Proposed Mitigation

Do not trust a user ID supplied through the request URL when determining
which user's allocation information may be accessed.

The application should obtain the authenticated user's ID from the server-side
session and use that identity when retrieving allocation records, rather than
using `req.params.userId` directly.

Where access to another user's object is intentionally supported, an explicit
server-side authorization check should verify that the authenticated user is
permitted to access the requested object.

**Control Location:** `app/routes/allocations.js` - `displayAllocations()`

## 6. Risk Assessment

Risk scoring uses a 3 × 3 model:

- Likelihood: 1 = Low, 2 = Medium, 3 = High

- Impact: 1 = Low, 2 = Medium, 3 = High

- Risk Score = Likelihood × Impact

ID | Threat                                                      | Likelihood | impact | Score | Risk |
---+-------------------------------------------------------------+------------+--------+-------+------+
T1 | Server-Side JavaScript Injection in Contribution Update     |  3         |  3     |  9    | High |    
T2 | NoSQL Injection in Asset Allocation Filtering               |  3         |  2     |  6    | High |
T3 | Session Fixation / Weak Session Management                  |  2         |  3     |  6    | High |
T4 | Insecure Direct Object Reference in Asset Allocations       |  3         |  2     |  6    | High |




### T1 Risk Justification

**Likelihood:** High (3). The contribution update functionality directly passes

user-controlled contribution values to `eval()`. The functionality is

available to an authenticated application user.

**Impact:** High (3). Successful injection could cause unintended server-side

JavaScript execution, affecting application integrity and potentially

confidentiality or availability

### T2 Risk Justification

**Likelihood:** High (3). The stock threshold is controlled by an authenticated
user and is directly interpolated into a MongoDB `$where` expression without
being restricted to the expected numeric value.

**Impact:** Medium (2). Successful manipulation of the query could expose asset
allocation and associated user information that the requesting user should
not receive. Certain injected expressions could also affect database
availability.

### T3 Risk Justification

**Likelihood:** Medium (2). Exploitation requires an attacker to obtain or
influence a usable session identifier before the victim authenticates.

**Impact:** High (3). Successful exploitation could allow the attacker to
impersonate an authenticated user and access sensitive account and retirement
information.

### T4 Risk Justification

**Likelihood:** High (3). The user ID is obtained directly from a URL
parameter and used to retrieve allocation information without verifying that
the requested ID belongs to the authenticated user.

**Impact:** Medium (2). Successful exploitation could disclose another user's
asset allocation and associated account information. However, the identified
functionality does not by itself demonstrate complete account takeover or
administrator-level access.

## 7. Threat-Control Mapping

### T1 - Server-Side JavaScript Injection

**Threat:** User-controlled contribution values are processed using `eval()`,

allowing the input to be interpreted as JavaScript.

**Affected Location:** `app/routes/contributions.js`,

`handleContributionsUpdate()`

**Proposed Control:** Remove `eval()` and process contribution inputs as

numeric data. Apply server-side validation to ensure that contribution values

have the expected numeric format and fall within an acceptable range.

**Control Location:** `app/routes/contributions.js`

**Verification:** Re-run the same security test used against the vulnerable

implementation after the fix and verify that the malicious input is rejected

or treated only as data rather than executable JavaScript.

### T2 - NoSQL Injection in Asset Allocation Filtering

**Threat:** User-controlled stock threshold input is directly incorporated
into a MongoDB `$where` expression, allowing the intended query logic to be
manipulated.

**Affected Location:** `app/data/allocations-dao.js`,
`getByUserIdAndThreshold()`

**Proposed Control:** Treat the threshold as numeric data rather than
executable query content. Parse and validate the threshold on the server and
reject values outside the expected range. Avoid constructing executable
MongoDB `$where` expressions from untrusted user input where possible.

**Control Location:** `app/data/allocations-dao.js`

**Verification:** Repeat the same allocation-filter security test after the
fix and verify that malicious/non-numeric threshold input is rejected and
cannot modify the intended query behavior.

### T3 - Session Fixation / Weak Session Management

**Threat:** The application does not regenerate the session identifier after
a successful login. Instead, the authenticated user's ID is assigned to the
existing session. If an attacker obtains or controls that session identifier
before authentication, the same session may remain valid after login and
could potentially be used to impersonate the authenticated user.

**Affected Location:** `app/routes/session.js`,
`handleLoginRequest()`

**Proposed Control:** Regenerate the session identifier immediately after
successful authentication using `req.session.regenerate()` before storing the
authenticated user's ID in the session. This invalidates the previous session
identifier and reduces the risk of session fixation.

**Control Location:** `app/routes/session.js`,
`handleLoginRequest()`

**Verification:** Log in and verify that the session identifier changes after
successful authentication. Also verify that the previous pre-authentication
session identifier cannot be reused to access authenticated functionality.

### T4 - Insecure Direct Object Reference in Asset Allocations

**Threat:** The asset allocation functionality trusts a user ID supplied
through the request URL. An authenticated attacker could manipulate this ID
and potentially retrieve allocation information belonging to another user.

**Affected Location:** `app/routes/allocations.js`,
`displayAllocations()`

**Proposed Control:** Do not trust the user ID supplied through the URL when
determining which user's allocation information should be retrieved. Obtain
the authenticated user's ID from the server-side session using
`req.session.userId` and use that value when retrieving allocation records.

**Control Location:** `app/routes/allocations.js`,
`displayAllocations()`

**Verification:** Log in as one user and attempt to request allocation
information using another user's ID in the URL. After the control is
implemented, changing the URL user ID should not allow access to another
user's allocation information.