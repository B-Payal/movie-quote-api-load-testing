# Load Test Report: Movie Quote API

## Test Configuration

* Tool: Artillery

* Total duration: 60 seconds (10s warm up, 30s ramp up, 20s sustain)

* Peak virtual users: 50 new arrivals per second

* Target URL: http://localhost:3001

* Recorded: 14-06-2026

## Baseline: Single-User curl Results

* GET /api/quotes/unpaginated: **7.9 ms**, **140537 bytes**

* GET /api/quotes?page=1&limit=20: **9.6 ms**, **2851 bytes**

* POST /api/favorites: **4.56 ms**

## Load Test Results: Unpaginated Endpoint

* Median response time: **4 ms**

* p95 response time: **8.9 ms**

* Throughput: **46 requests per second**

* Error rate: **0%**

## Load Test Results: Paginated Endpoint

* Median response time: **2 ms**

* p95 response time: **3 ms**

* Throughput: **50 requests per second**

* Error rate: **0%**

## Load Test Results: POST /favorites

* Median response time: **3984.7 ms**

* p95 response time: **6312.2 ms**

* Throughput: **39 requests per second**

* Error rate: **6.33%** (95 failed requests out of 1500)

## Comparison and Analysis

The load test results show that the paginated endpoint performs better than the unpaginated endpoint under concurrent traffic. The unpaginated endpoint recorded a **median response time of 4 ms**, a **p95 response time of 8.9 ms**, and a throughput of **46 requests per second**. In comparison, the paginated endpoint achieved a **median response time of 2 ms**, a **p95 response time of 3 ms**, and sustained **50 requests per second**, while maintaining a **0% error rate**.

The key reason for this improvement is the amount of data returned in each response. The unpaginated endpoint sends approximately **140,537 bytes** for every request, whereas the paginated endpoint returns only about **2,851 bytes**. The smaller payload requires less serialization, reduces memory usage, and minimizes network transfer time, allowing requests to complete more quickly and consistently.

The `POST /favorites` endpoint exhibited significantly worse performance under the same load. It recorded a **median response time of 3984.7 ms** and a **p95 of 6312.2 ms**, with **95 `ECONNREFUSED` errors**, resulting in an error rate of approximately **6.33%**. This suggests that the endpoint or its underlying database operations became a bottleneck under heavy concurrent traffic, causing requests to queue or be rejected.

## What p95 Means and Why It Matters

The **p95 response time** represents the maximum time within which **95% of requests complete**. For the unpaginated endpoint, the p95 was **8.9 ms**, meaning that almost all users received responses in under 9 milliseconds. The paginated endpoint performed even better with a **p95 of just 3 ms**, demonstrating highly consistent performance.

In contrast, the `POST /favorites` endpoint had a **p95 of 6312.2 ms**, meaning that 95% of users had to wait over six seconds or less for a response, while the remaining 5% experienced even longer delays. This highlights why p95 is often more useful than averages—it reflects the experience of nearly all users and quickly reveals performance degradation under load.

## Discovered Issues (optional)

The most significant issue observed during testing was the appearance of **95 `ECONNREFUSED` errors** while testing `POST /favorites`, resulting in an overall **6.33% error rate**. This indicates that the server or database was unable to accept some incoming connections when subjected to sustained load.

Although both GET endpoints completed successfully with **0% errors** and very low response times, the `POST /favorites` endpoint experienced severe latency, with response times increasing to nearly **7 seconds** in the worst case. These results suggest that write operations require further optimization, such as improving database performance, increasing connection pool capacity, or introducing batching or asynchronous processing for inserts.
