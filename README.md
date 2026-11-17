# Restful-Booker: API Testing with Postman and Newman

![Postman](https://img.shields.io/badge/Postman-FF6C37?logo=postman&logoColor=white)
![Newman](https://img.shields.io/badge/Newman-CLI-FF6C37)
![Node.js](https://img.shields.io/badge/Node.js-20+-339933?logo=nodedotjs&logoColor=white)

API test suite for [Restful-Booker](https://restful-booker.herokuapp.com/apidoc/index.html), a hotel booking REST API with token authentication and full CRUD.
It contains a **Postman collection with 24 test cases (11 positive, 13 negative)**, an environment file, assertions in every request, and a **Newman run that exports an HTML report**.

**Esha Israr** · SQA / Test Automation

---

## Highlights

- **24 API test cases** across health check, authentication, create, read, update and delete
- **Positive and negative scenarios**: wrong credentials, missing required fields, missing and invalid tokens, non-existent ids, repeated deletes
- **Assertions on every request**: status code, response body values, JSON schema, headers, and response time (checked for every request at collection level)
- **Request chaining**: the token and booking id from one request are saved to environment variables and reused by the next requests
- **Dynamic test data** generated in pre-request scripts, so every run uses unique data
- **Both auth methods covered**: token cookie and `Authorization: Basic` header
- **Newman run with an HTML report** (htmlextra), a JUnit XML report, and a **GitHub Actions** workflow

## Project structure

```
restful-booker-api-testing/
├── collections/Restful-Booker.postman_collection.json
├── environments/Restful-Booker.postman_environment.json
├── docs/
│   ├── API_Test_Cases.xlsx     test case sheet and findings sheet
│   └── API_Findings.md         API design observations
├── reports/                    Newman reports are written here
├── package.json                Newman and the HTML reporter
└── .github/workflows/newman.yml
```

## Environment variables

| Variable | Purpose | Set by |
| --- | --- | --- |
| `baseUrl` | API base URL | you (environment file) |
| `username`, `password` | Public demo credentials (`admin` / `password123`, published in the API docs) | you |
| `maxResponseTime` | Response time limit in ms (default 5000) | you |
| `token` | Auth token | TC_API_002 |
| `bookingId` | Id of the booking under test | TC_API_006 |
| `firstName`, `lastName`, `totalPrice`, `updated*`, `patched*` | Generated test data | pre-request scripts |

## How to run

### Option A: in Postman

1. Open Postman and choose **Import**. Import both files from `collections/` and `environments/`.
2. Select the **Restful-Booker (Production)** environment (top right).
3. Right-click the collection, choose **Run collection**, keep the order, and click **Run**.

### Option B: with Newman (command line)

Prerequisite: [Node.js](https://nodejs.org/) (LTS).

```bash
npm install
npm test
```

The report is written to **`reports/newman-report.html`** (open it in a browser), and a JUnit file to `reports/newman-junit.xml`.

To run Newman directly without the npm script:

```bash
npx newman run collections/Restful-Booker.postman_collection.json -e environments/Restful-Booker.postman_environment.json -r cli,htmlextra --reporter-htmlextra-export reports/newman-report.html
```

> The requests depend on each other (create, then read, update, delete), so run the collection **in order and as a whole**, not one request at a time.

## Report

> Add a screenshot of your Newman HTML report here: `docs/newman-report.png`
>
> `![Newman report](docs/newman-report.png)`

## Test cases

| Test case | Module | Method and endpoint | Type | Expected |
| --- | --- | --- | --- | --- |
| TC_API_001 | Health Check | `GET /ping` | Positive | 201: 201 Created |
| TC_API_002 | Authentication | `POST /auth` | Positive | 200: Token returned and saved to the environment |
| TC_API_003 | Authentication | `POST /auth` | Negative | 200: No token; reason 'Bad credentials' (the API answers 200, see findings) |
| TC_API_004 | Authentication | `POST /auth` | Negative | 200: No token; reason 'Bad credentials' |
| TC_API_005 | Authentication | `POST /auth` | Negative | 200: No token; reason 'Bad credentials' |
| TC_API_006 | Create Booking | `POST /booking` | Positive | 200: bookingid returned; booking data matches the request; schema valid |
| TC_API_007 | Create Booking | `POST /booking` | Negative | 500: Request rejected, no booking created (the API answers 500, see findings) |
| TC_API_008 | Create Booking | `POST /booking` | Negative | 500: Request rejected, no booking created |
| TC_API_009 | Read Booking | `GET /booking` | Positive | 200: Array of objects, each with a numeric bookingid |
| TC_API_010 | Read Booking | `GET /booking/{{bookingId}}` | Positive | 200: Booking data equals the data that was created; schema valid |
| TC_API_011 | Read Booking | `GET /booking` | Positive | 200: The list contains the booking created in TC_API_006 |
| TC_API_012 | Read Booking | `GET /booking/999999999` | Negative | 404: 404 Not Found |
| TC_API_013 | Update Booking | `PUT /booking/{{bookingId}}` | Positive | 200: All fields updated and returned |
| TC_API_014 | Update Booking | `PUT /booking/{{bookingId}}` | Negative | 403: 403 Forbidden, booking not changed |
| TC_API_015 | Update Booking | `PUT /booking/{{bookingId}}` | Negative | 403: 403 Forbidden, booking not changed |
| TC_API_016 | Update Booking | `PATCH /booking/{{bookingId}}` | Positive | 200: Names changed; the other fields keep the values set by the PUT |
| TC_API_017 | Update Booking | `PATCH /booking/{{bookingId}}` | Negative | 403: 403 Forbidden, booking not changed |
| TC_API_018 | Update Booking | `PATCH /booking/{{bookingId}}` | Positive | 200: additionalneeds updated; names keep the patched values |
| TC_API_019 | Update Booking | `GET /booking/{{bookingId}}` | Positive | 200: The stored booking contains every update |
| TC_API_020 | Delete Booking | `DELETE /booking/{{bookingId}}` | Negative | 403: 403 Forbidden, booking still exists |
| TC_API_021 | Delete Booking | `DELETE /booking/{{bookingId}}` | Negative | 403: 403 Forbidden, booking still exists |
| TC_API_022 | Delete Booking | `DELETE /booking/{{bookingId}}` | Positive | 201: 201 Created (the API's documented success code for DELETE) |
| TC_API_023 | Delete Booking | `GET /booking/{{bookingId}}` | Negative | 404: 404 Not Found |
| TC_API_024 | Delete Booking | `DELETE /booking/{{bookingId}}` | Negative | 405: 405 Method Not Allowed (see findings, REST convention would be 404) |

## Test design

- **Positive cases** check that each operation works and returns the right data.
- **Negative cases** check that invalid input and missing or invalid authentication are rejected, and that nothing changes when a request is rejected.
- **State verification**: after the PUT and PATCH requests, a GET confirms the changes were stored. After DELETE, a GET confirms the booking is gone.
- **Schema validation** (`pm.response.to.have.jsonSchema`) protects the structure of the booking response.
- Collection-level script checks the response time after every request.

## Findings: where the API differs from REST conventions

The suite asserts the API's actual behaviour, so it works as a stable regression baseline. These are recommendations, also listed in `docs/API_Findings.md`.

| ID | Endpoint | Observation | Expected per REST convention | Severity |
| --- | --- | --- | --- | --- |
| F-01 | `POST /auth` | Wrong or empty credentials return 200 OK with a 'reason' message instead of an error status | 401 Unauthorized | Medium |
| F-02 | `POST /booking` | A request with a missing required field returns 500 Internal Server Error | 400 Bad Request with a validation message | Medium |
| F-03 | `DELETE /booking/{id}` | A successful delete returns 201 Created | 200 OK or 204 No Content | Low |
| F-04 | `DELETE /booking/{id}` | Deleting a booking that does not exist returns 405 Method Not Allowed | 404 Not Found | Low |
| F-05 | `GET /ping` | The health check returns 201 Created | 200 OK | Low |
| F-06 | `POST /booking` | A successfully created booking returns 200 OK | 201 Created | Low |

## CI

`.github/workflows/newman.yml` installs Newman, runs the collection, and uploads the HTML report as a build artifact.

## Notes and limitations

- Restful-Booker is a **shared public sandbox** that resets its data every few minutes. The suite creates its own booking at the start and deletes it at the end, so it does not depend on existing data.
- The first request after a pause can be slow because the host sleeps when idle. If a response-time check fails, raise `maxResponseTime` in the environment.
- The credentials in the environment file are the public demo credentials from the API documentation. For a real system, keep secrets out of the repository.

## Possible improvements

- Data-driven iterations with a CSV or JSON file
- Contract tests with a shared OpenAPI schema
- Run against multiple environments (dev, staging) using separate environment files
- Pair with the UI automation project in [saucedemo-selenium-automation](https://github.com/EshaIsrarX01/saucedemo-selenium-automation)

## Contact

Esha Israr · [LinkedIn](https://www.linkedin.com/in/eshaisrar08) · [Email](mailto:eesha.israr08@gmail.com) · [GitHub](https://github.com/EshaIsrarX01)
