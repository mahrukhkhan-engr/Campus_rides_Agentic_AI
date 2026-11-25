# Campus Rides — Scenario-Based Answers

## 1. Two drivers tap "Accept" in the same second.
## Who gets the ride?

The first eligible driver whose acceptance transaction
successfully commits gets the ride.

The Ride Service performs an atomic conditional update
that requires:

- The ride status is PENDING.
- No driver is already assigned.
- The two-minute acceptance deadline has not expired.
- The driver is within 3 km of the pickup location.
- The driver has no other active ride.

Driver proximity is validated by the Location Service.
PostgreSQL row-level locking serializes competing
acceptance attempts.

A unique partial index on driver_id for ACCEPTED and
ENROUTE rides prevents driver double-booking.

The winning transaction assigns the driver and changes
the ride status to ACCEPTED. The second attempt is
rejected because the ride is no longer PENDING.

## 2. The rider cancels at the same moment a driver
## accepts. What happens?

Both acceptance and cancellation operate on the same
ride row using atomic database transactions.

Two outcomes are possible:

- Acceptance commits first:
  The ride becomes ACCEPTED. The subsequent cancellation
  changes it to CANCELLED and applies a $3 fee.

- Cancellation commits first:
  The ride becomes CANCELLED with no fee. The driver's
  acceptance attempt is rejected because the ride is
  no longer PENDING.

Row-level concurrency control ensures consistent state
transitions and correct cancellation fees.

Terminal rides cannot be cancelled again, preventing
duplicate cancellation charges.

## 3. The driver taps "Dropped off" and loses signal,
## so the app sends it again. Is the rider charged twice?

No. The drop-off operation is idempotent.

The Ride Service allows completion only when the ride
status is ENROUTE.

In a single database transaction, it:

1. Changes the ride status to COMPLETED.
2. Creates a payment event in the transactional outbox.

If the first request succeeds but its response is lost,
the retry detects the COMPLETED status and returns
success without creating another payment event.

If the first transaction fails, the retry can safely
complete the ride.

The Payment Worker processes the stored payment event
using a stable idempotency key:

ride_id + payment_type

A unique payment record and payment-provider idempotency
prevent duplicate charges, even during repeated retries.

Therefore, the rider is charged only once for the
completed ride.
