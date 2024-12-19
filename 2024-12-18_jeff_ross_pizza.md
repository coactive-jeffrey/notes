# 2024-12-18 Ross / Jeff: Pizza Tracker Approach

## Goals
 - Figure out the backend side of things
 - Query by
     - `dataset_id`
     - `ingest_job_id`
     - `job_type`
     - `customer_id`
 - Data Retention?

### Non-goals
- UX
- UI Page
- User (Customer) Experience

## Questions
- How "should" a customer query the status of their job?
    - Give them back an ID (e.g. `ingest_job_id`, per-request `request_id`)?
        - How to align everywhere?
    - I heard about some kind of "streaming" thing roughly defined as a job that you can start (by creating it) and then continuously add more and more assets to over an extended period of time. What's that?
- How to make this easy to use?
    - Align everyone on:
        - What ID to return back to the customer
        - What event data to store?
            - [x] `timestamp_dt`
            - [x] `org_id`
            - [x] `dataset_id`
            - [x] `request_id`
            - [x] `ingest_job_id`
            - [x] `details`
                - [ ] `coactive_asset_id`
                - [ ] `coactive_image_id`
                - [ ] `coactive_video_id`
                - [ ] `coactive_shot_id`
                - [ ] `coactive_audio_segment_id`
                - [ ] Future 
            - [ ] `event_type`
            - [ ] `entity_type`
        - Other
            - `error` ==> state/whatever
        - How to store event data?
            - [x] Kafka Topic
            - [ ] 
        - How to send an event
            - [ ] Automatic?
            - [ ] Easy-to-use library?
    - Make a `coactive-libraries` package?
        - How to do?
        - What about importing other `coactive-libraries` packages (e.g. `coactive-kafka`)?
- What pitfalls?
