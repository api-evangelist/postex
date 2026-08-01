---
name: Book and track a PostEx shipment
description: >-
  Book a cash-on-delivery shipment with PostEx and track it to delivery, using
  the merchant Order Integration API. Covers looking up the merchant pickup
  address and destination city, creating the order, and polling its status.
api: openapi/postex-order-openapi.yml
operations:
  - getMerchantAddress
  - getOperationalCityV2
  - createOrder
  - trackOrder
  - getUnbookedOrders
method: generated
generated: '2026-07-20'
---

# Book and track a PostEx shipment

Use the PostEx merchant Order Integration API to book a parcel and track it.
Base URL: `https://api.postex.pk/services/integration/api/order`.

## Authentication

Every request must send the merchant API token (from the PostEx merchant
dashboard) in the **`token`** request header. Omitting it returns HTTP 400
`Missing request header 'token'`. There is no OAuth flow.

## Steps

1. **Look up your pickup address** — call `getMerchantAddress`
   (`GET /v1/get-merchant-address`) to get the pickup address code you will
   reference when creating the order.
2. **Resolve the destination city** — call `getOperationalCityV2`
   (`GET /v2/get-operational-city`) and match the consignee's city to a PostEx
   operational city name. PostEx only delivers to cities in this list.
3. **Create the order** — call `createOrder` (`POST /v3/create-order`) with the
   consignee details, the pickup address code from step 1, the city name from
   step 2, your own order reference number, item count, and the
   cash-on-delivery amount. Save the returned tracking number.
4. **Track the shipment** — call `trackOrder`
   (`GET /v1/track-order/{trackingNumber}`) with the tracking number to poll the
   delivery status.
5. **Reconcile** — call `getUnbookedOrders` (`GET /v1/get-unbooked-orders`) to
   find orders you created that have not yet been booked into a load sheet.

## Conventions and error handling

- Requests and responses are JSON over HTTPS.
- Versions are path segments (`/v1`, `/v2`, `/v3`); use the latest create-order
  (`/v3/create-order`).
- There is **no idempotency-key header** — rely on your own unique order
  reference number to avoid duplicate bookings.
- Errors return a Spring Boot JSON envelope
  (`{timestamp,status,error,message,path}`), not RFC 9457 problem+json. A 400
  usually means a missing `token` header or an invalid field.
