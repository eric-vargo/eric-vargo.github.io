---
layout: post
title:  "Migrating Analytics Data into a Data Warehouse"
date:   2021-09-10 12:45:01
---
# WORK IN PROGRESS
## Creating an Analytics Data Warehouse to Improve Application Scalability and Data Integrity

The flagship application at one of my former employers was a streaming video presentation application (think of a CEO making an online presentation to the entire company using streaming video, slides, polls, surveys, Q&A, etc…).  It used event-sourcing to control the presentation flow and track all viewer data. This data was stored in a collection in a Mongo database.  The presentation uses it primarily to build out state of the presentation for both producers and viewers but we also source presentation analytics from it.  Initially, due to time constraints, we implemented a temporary solution for our analytics offering, knowing we would need to replace it with something more robust and scalable.  Using the event-sourcing data bits, we were building data structures on the fly and further processing the data by searching, filtering, sorting, then returning it to the end user.  We had to bring all the data into memory in order to filter out what was needed.  As a result of this, it did not perform well to our performance measurements and it was also very buggy.  We knew we needed to replace this shortly after going live with the application once we had more time.

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

