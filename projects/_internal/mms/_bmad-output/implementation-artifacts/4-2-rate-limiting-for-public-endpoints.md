# Story 4.2: Rate Limiting for Public Endpoints

Status: done

## Story

As a **service operator**,
I want rate limiting on public endpoints,
so that the API is protected from abuse and excessive traffic.

## Acceptance Criteria

1. **Given** middleware/rate-limit.ts is created using @upstash/ratelimit **When** a public endpoint receives requests **Then** a sliding window rate limiter allows up to 100 requests per minute per IP

2. **Given** a client exceeds 100 requests per minute **When** the next request arrives **Then** a 429 RATE_LIMITED error response is returned with the standard error envelope

3. **Given** rate limiting is configured **When** internal (authenticated) endpoints are called **Then** rate limiting is NOT applied to internal endpoints

4. **Given** the health check endpoint exists **When** GET /health is called after this story **Then** the response includes Redis connectivity status: `{ "status": "ok", "db": "connected", "redis": "connected" }`

5. **Given** Upstash Redis is unreachable **When** a rate limit check fails **Then** the request is allowed through (fail-open) to avoid blocking legitimate traffic

## Tasks / Subtasks

- [ ] Task 1: Create middleware/rate-limit.ts with Upstash Ratelimit
- [ ] Task 2: Add RATE_LIMITED error type to lib/errors.ts
- [ ] Task 3: Apply rate limit middleware only to public routes
- [ ] Task 4: Update /health endpoint to include Redis status
- [ ] Task 5: Write tests

## Dev Notes

- Use @upstash/ratelimit (already installed)
- Sliding window: 100 requests/minute per IP
- Fail-open on Redis errors
- Apply only to /v1/donations/public/* routes

## Dev Agent Record

### File List

- mms-api/src/middleware/rate-limit.ts (NEW - Upstash rate limit middleware)
- mms-api/src/middleware/rate-limit.test.ts (NEW - rate limit tests)
- mms-api/src/services/donations/routes.ts (MODIFIED - added rate limit to public route)
- mms-api/src/index.ts (MODIFIED - health check includes redis status)
- mms-api/src/index.test.ts (MODIFIED - updated health check tests for redis)
