START

│
├─► Q1: Does the caller need to know RIGHT NOW if the other service's action succeeded?
│
├─► YES ──► USE SYNCHRONOUS API
│            - Command or Query pattern
│            - Fail fast / rollback on error
│            - Strong consistency guaranteed for caller
│
│            Follow-up Event?
│              - Yes, if other services need to react
│              - Payload: Enriched Business Event (ID + key facts + metadata)
│              - Ownership: Authoritative service emits it
│
└─► NO ──► USE ASYNCHRONOUS EVENT (Pub/Sub)
               - Caller publishes fact ("X happened")
               - Subscribers act whenever they receive it
               - Eventual consistency is acceptable


               Payload Rules:
                 - Default: Enriched Business Event
                 - ID-only: if extra data is optional & cheap to fetch
                 - Never full entity unless same-domain internal

               Ownership Rules:
                 - Only the authoritative domain service emits the event
                 - Enrichment done by the event owner
                 - Consumers build own projections for cross-domain views

Example Walkthrough (E-commerce)


    Reserve stock before confirming order
        Need immediate success/failure → API (POST /inventory/reserve)
        On success, OrderService emits OrderPlaced event (enriched payload).


    Fulfillment updates order status to shipped
        Can be delayed a few seconds → Event (OrderShipped)
        Payload includes orderId, shipmentId, carrier, tracking number, shippedAt.


    Send marketing email after payment
        Doesn’t block payment flow → Event (PaymentReceived)
        MarketingService subscribes, no API call needed.


🛡 Governance Checklist


    Step 1: Ask the consistency question first — not “Is it critical?”
    Step 2: Decide integration style (API or Event)
    Step 3: Apply payload scope rules
    Step 4: Enforce event ownership & enrichment rules
    Step 5: Register schema & correlate IDs for traceability