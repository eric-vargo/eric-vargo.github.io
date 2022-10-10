---
layout: post
title:  "Task Outline for Teaching Coding"
date:   2019-02-11 16:02:54
---
We had a junior developer join our team who had virtually no prior development experience.  Not to go into too many details of how we ended up hiring her, but she had a background that translates well to software development and she showed a real knack for understanding development concepts so we decided to give her a shot. To begin from square one, I created a comprehensive outline on the tools she would need to learn and any skills needed including domain knowledge about our product.  I segmented the outline into a rough timeline to map out her first 90 days.  There was virtually no way to predict how quickly she would be able to pick all this information up, but the timeline estimate turned out to be pretty accurate.  Here is the outline:

Week 1
------
- get accounts setup
    - gmail
    - zoom
    - slack
    - github access
- meet team (again)
- expectations of a developer
    - what makes a good developer
    - be able to explain what you did
    - don’t guess, research what you don’t know and look to understand everything
    - comment the code - When commenting code explaining why you did something is more important than explaining what the code does.  It's easy to read the code and figure out what it does, but developers looking at your code generally don't have an idea why a certain piece of code was written.
    - Take your time!
- Product overview _(I'm generalizing product details to avoid mentioning any specifics)_
    - Product area 1
    - Product area 2
    - Product area 3
    - Go over basic use case from creation to completion
- Use the Product
    - basic use case from start to finish
- Friday Items
    - Use the app as a user on her own
        - basic use case from start to finish

Week 2
------
- Talk about code research from Week 1 to further stress importance
- Agile Development
    - https://agilemanifesto.org/
- Our Kanban approach to Agile development
    - https://en.wikipedia.org/wiki/Kanban_board
    - go over our workload
    - What are Sprints?
        - Jira
        - confluence docs
- Walkthrough completing a small bug from start to finish
    - https://en.wikipedia.org/wiki/Use_case
    - give a small task and walk her through the dev process
    - planning what to do
    - using git to branch, etc…
        - talk a lot about branching and how it works
    - pull request
- Build Pipeline (brief overview)
    - Jenkins
    - what happens after you merge a PR
- Research items
    - git
        - branching
    - javascript
    - web pack / npm

Week 3
------
- completed bug with assistance
    - let her drive, but stick with her through the process
- REST API stuff
    - Postman
    - Http Status codes
        - https://httpstatuses.com/
    - coincides with calendar task to reduce fields
- Stack traces and debugging
    - https://harrymoreno.com/2017/02/25/how-to-read-a-javascript-stack-trace.html
    - source maps
    - what is "uglified code" and why do we do it
- General Javascript Research 
    - Again, stress the importance of research and how it will be the foundation of all her future development
    - arrow functions
    - classes
    - javascript inheritance
    - axios
    - debugging
        - source maps
        - web pack
        - execution context

Week 4 (leading up to Day 30)
------
- React Routing research
    - https://blog.pshrmn.com/entry/how-single-page-applications-work/
    - https://medium.com/@pshrmn/a-simple-react-router-v4-tutorial-7f23ff27adf
    - will coincide with registration stuff
- Material Design
    - https://material-ui.com/
- Styled Components research
    - https://www.styled-components.com/
- Starting on some small product tasks by the end of the week

Day 30
------
- should be comfortable completing a small bug from start to finish, autonomously
    - tasks needed to support any new demo requirements?
- Progess Notes
    - showed some good initiative in thinking about how form validation should work for an aspect of the product

Day 60
------
- Exception handling
    - using axios
- Research Unit tests
    - TDD
    - Unit test for video player URLS could tie in nicely with VideoJs learning
- Create a bug with unit tests
- HLS Video
    - Learn how the application uses videojs
        - https://docs.videojs.com/
    - ABR
    - different video formats
- Complete a bug on a more advanced area of the application, with assistance
    - Complete tasks related to production readiness?
- Websockets

After 90 days
-------------
- Present to the team an explanation on how an aspect of the product works
    - lots of javascript docs reading 
    - React docs
    - styled components docs
        - CSS docs
    - Update Readme or add comments to code
- completing Jira tasks w/ unit tests autonomously
- Bonus:
    - creating new functionality

