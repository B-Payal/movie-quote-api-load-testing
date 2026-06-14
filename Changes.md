# Changes.md

## Manual Investigation Notes

After running the Movie Quote API and testing the endpoints manually, I observed the following behaviors and potential issues.

### 1. Missing CORS Configuration

* The server does not configure CORS middleware.
* Requests from a frontend running on a different origin may fail with **"No Access-Control-Allow-Origin"** errors.
* This would prevent browser-based applications hosted on another port or domain from accessing the API.

---

### 2. Unpaginated Endpoint Returns a Very Large Payload

**Endpoint:** `GET /api/quotes/unpaginated`

* Returns all 1,000 movie quotes in a single response.
* The response size is significantly larger than the paginated endpoint.
* Under load, repeatedly serializing and transmitting such a large payload can increase CPU usage, memory consumption, and network latency.
* The endpoint also forces **Transfer-Encoding: chunked** instead of providing a `Content-Length` header, which may negatively impact performance for large responses.

---

### 3. Pagination Metadata Bug

**Endpoint:** `GET /api/quotes?page=1&limit=20`

* The `totalPages` calculation is incorrect when the total number of quotes is exactly divisible by the page size.
* For example:

  * Total quotes = 1000
  * Limit = 20
  * Expected `totalPages` = **50**
  * Actual `totalPages` = **51**
* This causes incorrect pagination metadata and may lead clients to request non-existent pages.

---

### 4. Blocking Operation in POST /favorites

**Endpoint:** `POST /api/favorites`

* The endpoint intentionally blocks the Node.js event loop for approximately 50 ms using a busy-wait loop.
* Because Node.js runs JavaScript on a single thread, this synchronous delay prevents other requests from being processed during that time.
* During load testing, this can significantly increase response times and reduce throughput.

---

### 5. Missing Input Validation

**Endpoint:** `POST /api/favorites`

* The API accepts any request body without validating `quoteId`.
* Requests such as:

```json
{}
```

or

```json
{
  "quoteId": null
}
```

are still accepted and stored.

* Invalid or missing values should ideally return a `400 Bad Request` instead of being saved.

---

## Manual Baseline Observations

* The paginated endpoint returned a much smaller payload than the unpaginated endpoint.
* The unpaginated endpoint required considerably more data transfer, making it less efficient under concurrent load.
* The POST endpoint worked correctly for valid requests but became significantly slower during stress testing because of the synchronous blocking delay.

---

## Overall Conclusion

The API functions correctly for basic usage but contains several intentional implementation issues that affect reliability and scalability. The most significant performance concerns are the large unpaginated responses and the blocking code inside `POST /favorites`, both of which become apparent during load testing. Additionally, missing CORS support, incorrect pagination metadata, and lack of input validation reduce the robustness of the application and should be addressed before production deployment.
