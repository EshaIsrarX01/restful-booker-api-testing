# API findings: Restful-Booker

Design observations from testing, compared with common REST conventions. The test suite asserts the API's **actual, documented behaviour**, so the suite is a stable regression baseline. These findings are recommendations, not blockers.

| ID | Endpoint | Observation | Expected per REST convention | Severity | Test cases |
| --- | --- | --- | --- | --- | --- |
| F-01 | POST /auth | Wrong or empty credentials return 200 OK with a 'reason' message instead of an error status | 401 Unauthorized | Medium | TC_API_003, 004, 005 |
| F-02 | POST /booking | A request with a missing required field returns 500 Internal Server Error | 400 Bad Request with a validation message | Medium | TC_API_007, 008 |
| F-03 | DELETE /booking/{id} | A successful delete returns 201 Created | 200 OK or 204 No Content | Low | TC_API_022 |
| F-04 | DELETE /booking/{id} | Deleting a booking that does not exist returns 405 Method Not Allowed | 404 Not Found | Low | TC_API_024 |
| F-05 | GET /ping | The health check returns 201 Created | 200 OK | Low | TC_API_001 |
| F-06 | POST /booking | A successfully created booking returns 200 OK | 201 Created | Low | TC_API_006 |

> Restful-Booker is a practice API built to be tested, and several of these behaviours are intentional quirks. They are listed here to show how deviations from REST conventions are identified and documented.
