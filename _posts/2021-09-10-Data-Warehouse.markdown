---
layout: post
title:  "Migrating to a Data Warehouse without Disrupting a Live Application   
or: How I Learned to Stop Worrying and Love Feature Flags"
date:   2021-09-10 12:45:01
---

The flagship application at one of my former employers used event-sourcing to control application flow by tracking all user movements and application state changes. The data was stored in a collection in our main Mongo database cluster.  Not only did this data track user movements, but we also sourced analytics from it for viewing at a later point in time.  It resembled what you might see as unstructured data in a Data Lake, so schemas had to be built on the fly before filtering any data
users requested.  This was a trade-off we made early on while creating the MVP so we could reduce time needed to go to market.  

We knew we would eventually need a more scalable solution to support all current and future reporting requirements.  Adding to the problem was we had an application that was live and generating revenue, so any migration could not disrupt our current clients.  We needed a solution that could run alongside the current application and allow us to change over to the new warehouse when it was fully tested and ready to be released.

## Getting the MVP to Market Quickly; Trade-offs

Due to our need to get a working application in a production environment, we implemented this temporary solution for our analytics offering, knowing we would eventually need to replace it with something that was more robust and scalable.  Using the event-sourcing data bits, we were building data schemas on the fly and further processing the data by searching, filtering, sorting, then returning it to the end user.  We had to bring all the data into memory in order to filter out what
was needed.  As a result of this, we could easily theorize memory limits and run tests to validate/dispel our concerns.  The application did not perform well to our performance measurements and it was also very buggy.  We knew we needed a better analytics and reporting product in order to compete.

## Outline of Our Plan

To solve these issues, I spearheaded a project to create an analytics data warehouse that would allow us to scale our business by supporting reporting that far exceeded our client base at the time.  This is the outline of the plan I created:

_Planning Phase_
* Settle on technologies 
    * SQS, Akka Actors, JDBI, briefly considered AWS Aurora but settled on MySQL on RDS
* Create estimates for each phase, highlighting inflection points requiring more developer attention
    * Document sensitive items that could become problematic and affect timelines
* Data Warehouse ERD design: Facts and Dimensions
    * [https://en.wikipedia.org/wiki/Star_schema](https://en.wikipedia.org/wiki/Star_schema)

_Development and Testing Phases (concurrent)_
* I created an end-to-end implementation for 1 piece of data to be the model “happy path” for developers to use as a reference
* Set development milestones for the 4 main application data points
    * Included testing with migrated production data in dev environments
* Testing behind feature flag for the 4 application data points
    * Testing was integrated with the dev phase as code for a single data point was dev-complete
    * Move to production (behind feature flag) once a single data point was deemed prod-ready
        * Allowed testing with live data behind a feature flag

_Release Phase (feature flag activation)_
* Data Migration and Data Reconciliation scripts
* Release manual testing, diagonal slice of key prod data
* Sunsetting current analytics code

I estimated we could complete the project and be in a production-ready state in about 6 months with a margin of error of a few weeks.  There weren’t any major unknowns for this project, such as using new libraries or integrating with a 3rd party vendor for the first time, so I felt that our estimates would be fairly close to exact.  Our main approach was to create end-to-end modules as quickly as possible so we could iterate over it and perform extensive testing.  I knew the testing phase and short, multiple feedback loops would be crucial to this project due to potential issues that could arise in the database.

## Development Phase

At a high level, the basic flow for storing data into the new warehouse would involve filtering through our event-sourcing data stream in the live application, and sending the relevant data to the new warehouse via a messaging queue.  The warehouse would then be responsible for ensuring data integrity during this process.  Any new code that was written would be adjacent to existing code, meaning our current code should not need any modifications (think of an Observer pattern).
This “add-on” concept was important to point out during the design phase so the development team would understand how to think about the new project.  

The messaging flow to the new warehouse should be something we could conceivably turn on and off.  We wanted the flexibility to test intermittently in production and also to possibly turn the feature off if we found bugs and decided it wasn’t quite ready for demos or other internal presentations.  Because of this, it was a perfect candidate for a Feature Flag.  We knew we needed this flexibility and that implementing Feature Flags (on the front-end for display and back-end for
ingestion) would be crucial to our testing phases and also to the release timeline.

In addition to the usual amount of hurdles in these types of projects, the front-end application was responsible for creating the data stream, so we didn’t have the luxury of creating perfect inputs from scratch.  I didn't want the front end to have to change anything unless we absolutely needed it in order to have full data integrity.  This decision limited disruptions to the product roadmap and would keep the spirit of the new development of being an “add-on”.

During project planning, I did not include myself as a development resource and that decision allowed me to “roam” around the project doing whatever was needed to fill in any blanks and keep the project running smoothly.  I believe this hands-on approach is crucial for any Principal-level technical leader to be successful.  The Ivory Tower concept is dead…roll up your sleeves and start hitting some keys!  The development team was mostly senior but they didn’t have a lot of experience
working on projects of this magnitude.  Not being tethered to any specific tasks allowed me to perform extensive edge case testing, saturation testing, and coding connective tissue that was missing regarding race conditions in the database, exception handling, etc…  After addressing some of the initial development work, I could then show developers what was missed and what to focus on during future development tasks.  This allowed the development phase to run smoothly, on
schedule, and also set the standard for development on future projects in terms of the level of detail required and how to focus on testing to support quick feedback loops.

## Testing and Migration

For the impending migration, we wrote a tool to migrate all our historical data into the new warehouse.  We knew we needed this tool for the migration, but during development, we also realized it would be very useful to have the tool operate on a smaller scale in our application.  This allowed us to create test data once and use it multiple times by wiping it out and re-running the migration tool.  With the aid of these tools, we could extensively test messages coming into the warehouse
that were operated on “out of order” due to the asynchronous nature of messaging and our application data  (We had create, edit, and delete messages for all our data types that could be received out of order during a migration run).  This process uncovered a lot of race conditions that we were able to address with great efficiency using the new tools we created.

I wanted to achieve a few things in the database: idempotency and to not use any locking.  This would require a lot of testing of the migration tool.  We would compare the results of multiple runs to see if we would get different results in the database.  If we saw differences across migration runs, we knew we had a race condition to address.  While we were writing the migration and reconciliation code, we actually discovered some bugs in the old system.  The new system was already more
reliable!

## Go Live!

When we were getting closer to going live, we opened up testing in production to stakeholders within the organization.  The feature flags allowed everyone to test using production data by comparing the old data APIs and the new APIs for the warehouse.  Our analytics reports were, at the time, still hitting the previous APIs for their data, but users could turn on the feature flag by setting a variable in the javascript console.  That would allow them to then get analytics data
from the new APIs.  They could test this however they needed which often included side by side comparisons of the old and new data.  Once everyone was happy with their testing, we were able to switch over to using the new warehouse with minimal effort that amounted to removing the front end feature flag code.  The warehouse was now live!

## Reflections

We ended up taking closer to 7 months to complete the project, mostly because the developers were a bit rusty in using SQL (myself included) and that phase took a bit longer than expected.  Two areas that set us back were in processing messages out of order (asynchronously) and the more complicated select statements that supported reporting.  That was a reasonable setback to me since our main application only uses Mongo and we hadn’t used SQL this complex for a few years.  

The measurement metric for success was simple:  Working software with no major bugs.  The use of a feature flag allowed us to run extensive testing in the production warehouse long before we made it visible to our users.   Because of this, after going live, we saw zero bugs in production for over a year since the go-live date.  Nothing says success better than that!

