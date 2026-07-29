---
title: "Week 2 Worklog"
date: 2026-06-22
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
---

### Week 2 Objectives:

* Set up a unified database system for the microservices.
* Develop the authentication and authorization service (Identity Service) using Java Spring Boot.
* Develop the course content management service (Course Service) using Java Spring Boot.

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 1 | - Design the table structure for Identity Service and Course Service <br> - Define the database schema using Prisma ORM and initiate migrations | 22/06/2026 | 22/06/2026 | Prisma ORM Schema Reference |
| 2 | - Develop registration, login (Access/Refresh Token), and logout (Token blacklisting via Redis) features in the Identity Service | 23/06/2026 | 23/06/2026 | Spring Security & JWT Guidelines |
| 3 | - Integrate BCrypt password hashing and configure automatic initialization of default roles (`STUDENT`, `INSTRUCTOR`, `ADMIN`) | 24/06/2026 | 24/06/2026 | Spring Boot Security Config |
| 4 | - Build REST APIs to manage course lifecycle in the Course Service (Categories, details, sections, and lessons) | 25/06/2026 | 25/06/2026 | Course Management API Docs |
| 5 | - Locally test all authentication and course management APIs under the local environment | 26/06/2026 | 26/06/2026 | Postman API Testing Guide |

### Week 2 Achievements:

* Completed detailed DB schema design and managed migrations consistently through Prisma ORM.
* Completed the Identity Service with full Authentication & Authorization functionality secured via JWT and Redis Blacklist.
* Completed REST APIs for the Course Service satisfying category and lesson management requirements.
* Successfully tested all major endpoints of the two services locally (Local Testing).
* **Early Containerization**: Authored optimized Dockerfiles and launched containers for the core services under the local Docker environment to ensure independent runtime configuration.

![Docker Image List](/images/1-WorkLog/docker-identity-service-2.png)
*List of Docker Images successfully built on the local machine for microservices.*

![Docker Container Running](/images/1-WorkLog/docker-identity-service.png)
*Launching the Identity Service container inside the local Docker Desktop interface listening on port 8080.*

