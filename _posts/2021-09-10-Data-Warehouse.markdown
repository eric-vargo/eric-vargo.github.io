---
layout: post
title:  "Migrating to an Analytics Data Warehouse without Disrupting a Live Application in Production 
or: How I Learned to Stop Worrying and Love Feature Flags"
date:   2021-09-10 12:45:01
---
The flagship application at one of my former employers used event-sourcing to control application flow by tracking all user movements and application changes. This data was stored in a simple collection in a Mongo database because that was our main database server and it was very easy to query and retrieve all needed data.  Not only did this data track user movements, we also source analytics from it for viewing at a later point in time.  This was a trade-off we made early on while creating the MVP so we could reduce time needed to go to market.  We knew we would eventually need a data warehouse to support all current and future reporting requirements.  The problem we had was we had an application that was live and generating revenue so any migration could not disrupt the business.  We need a solution that could run alongside the current application then would allow us to change over to the new warehouse when it was fully tested and ready to release.

# WORK IN PROGRESS
## Creating an Analytics Data Warehouse to Improve Application Scalability and Data Integrity

To solve these issues, I spearheaded a project to create an analytics data warehouse that would allow us to scale our business by supporting presentation analytics that far exceed our current viewership.  This is the outline of the plan I came up with (leaving out a lot of the details for the sake of brevity):

### Planning Phase

* Settle on technologies (SQS, Akka, briefly considered AWS Aurora but settled on MySQL on RDS)
* Create estimates for each phase, highlighting inflection points requiring more developer attention
* Data Warehouse design: Facts and Dimensions

### Development and Testing Phases (concurrent)

* I created an end-to-end implementation for 1 piece of data to be the model happy path for developers
* Set development milestones for the 4 main application data points
* Includes testing with migrated production data in dev environments
* Testing behind feature flag for the 4 application data points
* Testing was integrated with the dev phase as code for a single data point was dev-complete
* Move to production (behind feature flag) once a single data point was deemed prod-ready

### Release Phase (feature flag activation)

* Data Migration and Data Reconciliation scripts
* Release testing, diagonal slice of key prod data
* Sunsetting current analytics code

