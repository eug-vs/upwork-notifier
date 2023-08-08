# Upwork notifier

[Upwork API refrerence](https://developers.upwork.com/?lang=node)

Upwork rate limits:

- 1req / 1.5sec
- 60req / min (?)

We potentially have multiple Upwork accounts, so limit applies to each one independently. Further reasoning in based on one account.

## Achitecture outline

1. The job is scheduled to run periodically with a set interval (probably via Upstash crons)
2. Job triggers `collect` endpoint (lambda), which performs the following steps:
   1. Authorization to Upwork API
   2. Fetching the list of rooms
   3. (optionally) Filtering out uninsteresting rooms
   4. Triggers the `fetchRoom` recursive-like endpoint (lambda), providing auth data and the list of rooms to be fetched:
      1. Waits for rate-limit to be respected (via internal rate-limiter connected to Redis, or receives last request timestamp via params)
      2. Fetches the first chat room from the list
      3. Stores the data in some ephemeral storage (probably Redis or SQL DB with TTL)
      4. Compares the old data to the new data, triggering telegram notification if needed
      5. Calls `fetchRoom` endpoint with the list of remaining rooms
