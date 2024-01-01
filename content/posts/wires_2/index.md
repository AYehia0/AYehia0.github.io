---
title: "Wires #2: System Design"
date: 2023-12-29T13:55:42+02:00
draft: true
toc:
  enable: true
  auto: false
share:
  enable: true

authors: []
description: ""

tags: []
categories: []
series: []

hiddenFromHomePage: false
hiddenFromSearch: false

featuredImage: "final.png"
featuredImagePreview: "featured-image.jpg"
---

In this blog, we will make the system design for wires, based on the previous requirements and thoughts we gathered about the project.

<!--more-->

## Documented Business Requirements
When creating a decent system design, a documented business requirements (aka SRD: System Requirements Document) is used to provide clear and detailed description of the system, and it confirms that our system meets all the needs of all the stakeholders.

SRD highlights the functional and non-functional requirements of the system. Let's recap what we have achieved so far.

<b>Non-Functional Requirements:</b>
- Performance: the system should handle high volume of simultaneous package tracking requests.
- Scalability: the system should be scalable to handle the increase of drivers, packages and customers/recipients.
- Reliability: minimal downtime and errors. That's a whole big concept [Designing Data Intensive Application].
- Security: it's not important for our project to be secured but will be extendable for adding security layers later.
- Usability: Easy and intuitive user interface.

<b>Functional Requirements:</b>
- User Roles: 
  - Driver, Package Sender(Amazon, or other company), and Recipient/User.
  - Admin which is a user that sees all the traffic.
- Package Tracking: 
  - System should provide realtime tracking of packages
  - Recipients should be able to view the current status and location of their packages in realtime.
- Driver Assignments: assign packages to drivers.
- Notifications: Notify/Update recipients when status changes of package delivery.

### Design Flow
Usually you should know what flow you're going to take when creating such a project. In our case (Small project), this flow might actually work as the requirements are pretty simple. 

{{< admonition tip "Remember" >}}
Continuous iteration and updating the documentation you progress in the project, help to ensure that the system meets all requirements.
{{< /admonition >}}

We're going to follow this flow to produce a database design (since it's one important aspect in the design #check below) that satisfies the requirements.
{{< mermaid >}}graph LR;
    A[Product Requirement Doc] -->B(Abstract Models/Concepts)
    B --> C[Data Definitions]
    C --> E[Objects]
    E --> J[Database]
{{< /mermaid >}}

Converting a Product Requirement Document (PRD) to abstract concepts involves distilling the detailed and specific requirements outlined in the PRD into higher-level and more generalized ideas or concepts. This process helps to identify the core principles, functionalities, and goals of the product without getting into the nitty-gritty details. The purpose is to create a conceptual framework that can guide the overall design and development of the product. Before going that way it's important to decide what are the key aspects of design that will used in the conversion.

## Design Aspects
The focus is on translating the abstract concepts and high-level requirements into concrete design considerations. This step is critical for guiding the actual implementation of the product.

1. User Experience Design: it's predefined in a wireframe from the previous devlog#1 (Just a general Idea of how things will look like).
2. System Architecture: Microservices vs. Monolith. We'll discuss this topic further more.
3. Scalability and Performance: Load balancing, caching and how to identify scalable components.
4. Security: Authentication and Authorization (not today).
5. Data Management: Validation and Storage.
6. Testing: Define a strategy to test and validate the system.
7. Documentation: API and System Docs.
8. Monitoring and Analysis: Performance, Load, Storage and overall system Monitoring.
9. Deployment: CI/CD for automated deployment.

## System Design: Abstract Version

{{< image src="overall_design.png" position="center" style="border-radius: 8px;" caption="A High Level Abstract Design With Questions that need answers!" >}}

From the proposed design I created, it descripes how some components interact with each other:-

{{< mermaid >}}graph LR;
    M(Draw Map) -->A(Spawn Objects)
    A --> X[Drivers]
    A --> Y[Recipients]
    A --> C(Initial Simulation)
    C --> E(Start Realtime Simulation)
{{< /mermaid >}}

1. Draw the map: the server sends the map to the frontend, where it's it's created!
```json
{
    coords: [[299,303], [29,32], [203, 302]] // ofc it's not gonna look like that idk!
}
```
2. Spawn the Objects (Drivers and Recipients): return locations of objects.
  - `createDrivers(num_drivers, grid_size)`: create number of drivers in that grid size/location.
  - `createRecipients(num_recipients, grid_size)`: create number of recipients in that grid size/location.
  - `createOrders(user_id, location, grid_size)`: create orders/packages in some locations bounded by the grid size.
3. Initial Simulation: get best ideal routes with order of packages.
  - `calculateIdealPath(drivers, packages)`
4. Start Simulation: here is the good part, where everything happens.

Notes, We can just consider all these steps above are part of the Simulation engine as the key components of the visualization engine are Recipients and Drivers, where periodical decisions are made in infinite loop:-
- Spawning and despawning.
- Ideal route searching.
- Location updating.

## Takes on the design
Since there are many instances of the drivers and packages, running simultaneously within the same process it's a good idea to have a lightweight visualization process and to achieve so, it's important to make the simulation updating in a predefined window of interval. And blocking the main process isn't a good thing.

Processes which are intensive:
- Generating destinations and Finding optimal routes.
- Connecting packages with drivers.

A queue might be handy in an architecture like that; when a users requests a package delivery or any other service, the response is awaited and it that time something else could be happening like driver might be delivering something else (idk).
## Database Design
Usually the API Design comes before this one but for the sake of simplicity, I am going with the database first.
{{< image src="db_design.png" position="center" style="border-radius: 8px;" caption="Very Simple Database Design" >}}

### Choosing Database
###  Sql vs NoSql
Probably NoSql databases would suit this project the most due to its flexibility and the write throughput, but I want to go with the best of all databases (PostgreSql - `I want to get better with SQL`).

Note that this system is a write heavy system, with (idk) read less ? throughput. 
## Network Protocols
While Normal TCP/RESTful is a valid approach, WebSocket Protocol seems best to be used here due to:
- Bi-Directional Communications.
- Low Latency.
- It's ideal for realtime applications.
- Idk about the scalability.

## API Design
If you reached this point, I just want to finish this blog and start working on the project (I want to sleep too, it's 2:36AM, and I have been thinking about this design for 3 days).
- `/drivers` : spawn drivers with more info from the simulation/visualization engine.
- `/recipients`: spawn recipients with more info from the simulation/visualization engine.
- `/status`: notification updates

## Final Design
There are lots of things that can be added to this design, later on we're going to extend it.

{{< image src="final.png" position="center" style="border-radius: 8px;" caption="Final Design!" >}}

## Notes
- This design is an initial in-complete design that will just works, can be extended with more features.
- Idks in this blog show things that I am not sure about.
- This isn't the best design in the world but it meets the requirements (I think lol).

## Resources
- [System Design For Beginners - freeCodeCamp](https://www.youtube.com/watch?v=m8Icp_Cid5o)
