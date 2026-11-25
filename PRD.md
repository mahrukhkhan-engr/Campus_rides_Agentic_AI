# Campus Rides — Product Requirements Document

## 1. Goals

- Enable fast and reliable campus ride matching.
- Connect riders with available drivers within 3 km
  of the pickup location.
- Ensure consistent ride assignment, transparent pricing,
  and safe payment processing.
- Support intermittent network connectivity.

## 2. Non-Goals

- Multi-city operations.
- Surge pricing and promotional discounts.
- Driver ratings and reviews.
- Ride sharing or multiple passengers per booking.

## 3. User Stories

- As a rider, I want to enter pickup and drop-off
  locations and request a ride.
- As a driver, I want to view nearby ride requests
  and accept an available ride.
- As a rider, I want to see the assigned driver's
  name, car information, and distance after acceptance.
- As a driver, I want to mark a passenger as picked up
  and dropped off.
- As a rider, I want transparent pricing and protection
  against duplicate charges.
- As a rider, I want to cancel a ride and know
  the applicable cancellation fee.

## 4. Business Rules

1. Only one driver can be assigned to a ride.
2. A driver can have only one active ride at a time.
3. A rider can have only one open request at a time.
4. Only available drivers within 3 km of the pickup
   location can receive and accept the request.
5. The first successful acceptance wins.
6. Unaccepted requests expire after two minutes,
   displaying "No drivers nearby".
7. Cancellation before acceptance is free;
   cancellation after acceptance costs $3.
8. Fare = $2 + ($1 × ride distance in km).
   Fare is calculated and fixed at request creation.
9. The fare is charged only after successful drop-off.
10. Duplicate requests must not cause duplicate
    assignments, state transitions, or payments.

## 5. Operational Requirements

- Support 300 drivers and 2,000 ride requests per hour.
- Receive driver location updates every five seconds,
  approximately 60 updates per second.
- Support safe retries and state synchronization
  after network reconnection.

## 6. Measurable Success Criteria

- Zero duplicate driver assignments.
- Zero driver double-booking incidents.
- Zero duplicate payment charges.
- All unaccepted requests expire after 120 seconds.
- 100% accuracy of fare and cancellation calculations.
- Sustain the specified request and location workload.
- Target acceptance response time of one second
  or less under the specified workload.
