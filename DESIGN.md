# Campus Rides — Technical Design

## 1. System Components

- Rider/Driver Apps: HTTPS APIs, WebSocket notifications,
  retry support and state synchronization after reconnection.
- API Gateway: Authentication, authorization and rate limiting.
- Ride Service: Request creation, matching and ride lifecycle.
- Location Service: Driver location updates every 5 seconds;
  PostGIS proximity queries within 3 km of pickup.
- Payment Worker: Reliable, idempotent payment processing.
- Expiry Worker: Expires requests at their two-minute deadline.
- PostgreSQL: Authoritative storage and atomic transactions.
  Redis: Optional location and availability cache.

## 2. Data Model

| Entity | Key Fields |
|---|---|
| Users | id, name, role |
| Drivers | id, user_id, car_info |
| Locations | driver_id, coordinates, updated_at |
| Rides | id, rider_id, driver_id, pickup, dropoff, distance_km, fare_cents, status, created_at, expires_at |
| Payments | id, ride_id, type, amount, status, idempotency_key |
| Outbox | event_id, ride_id, event_type, processing_status |

Ride states:
PENDING → ACCEPTED → ENROUTE → COMPLETED

PENDING → NO_DRIVER (timeout)

PENDING / ACCEPTED / ENROUTE → CANCELLED

## 3. API Endpoints

| Endpoint | Purpose |
|---|---|
| POST /rides | Create request and calculate fixed fare |
| GET /rides/{id} | Retrieve ride state and assigned driver details |
| GET /drivers/nearby-requests | Return eligible pending requests within 3 km |
| POST /drivers/location | Update driver location |
| POST /rides/{id}/accept | Atomically assign eligible driver |
| POST /rides/{id}/cancel | Cancel ride and determine fee |
| POST /rides/{id}/pickup | ACCEPTED → ENROUTE |
| POST /rides/{id}/drop-off | ENROUTE → COMPLETED and queue payment |

## 4. Business Rule Enforcement

| Rule | Enforcement |
|---|---|
| One driver per ride | Atomic conditional UPDATE on pending, unassigned ride |
| One ride per driver | Unique partial index on driver_id for ACCEPTED/ENROUTE rides |
| One open request per rider | Unique partial index on rider_id for PENDING rides |
| Driver within 3 km | Location Service filters requests and revalidates eligibility at acceptance |
| Two-minute expiry | expires_at = created_at + 120 seconds; accept rejects expired requests; worker marks NO_DRIVER and notifies rider |
| Cancellation | Row-level transaction: PENDING = free; ACCEPTED/ENROUTE = $3; terminal states cannot be cancelled again |
| Fixed fare | Backend computes $2 + $1/km at creation; stored fare remains immutable |
| Driver information | After acceptance, rider receives driver name, car information and current distance |
| Single payment | Unique payment key, transactional outbox and provider idempotency |

## 5. Concurrency and Reliability

- Acceptance uses a conditional database update and the
  unique driver index in one transaction. Conflicts are
  rejected without changing the ride.
- Acceptance and cancellation serialize on the same ride
  row. The first valid committed transition determines
  the outcome and cancellation fee.
- Drop-off is allowed only from ENROUTE. Completion and
  payment-outbox insertion occur in one transaction.
- Payment worker retries using a stable key:
  ride_id + payment_type. Duplicate events cannot
  produce duplicate charges.
- Authenticated users can modify only their own rides.
- Clients retry safely and fetch authoritative ride
  state after network reconnection.

## 6. Scalability

300 drivers / 5 seconds = 60 location updates/second.

2,000 requests/hour ≈ 0.56 ride requests/second on average.

Use indexed geospatial queries, database connection
pooling, asynchronous notifications and payment workers.
Load and failure testing are required before production.
