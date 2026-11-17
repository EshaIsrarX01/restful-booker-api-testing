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

| Variable | Purpose |
| --- | --- |
| `baseUrl` | API base URL | 
| `username`, `password` | Public demo credentials (`admin` / `password123`, published in the API docs) |
| `maxResponseTime` | Response time limit in ms (default 5000) |
| `token` | Auth token |
| `bookingId` | Id of the booking under test |
| `firstName`, `lastName`, `totalPrice`, `updated*`, `patched*` | Generated test data |

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
