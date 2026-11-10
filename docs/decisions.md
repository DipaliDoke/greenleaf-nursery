# GreenLeaf Nursery Architecture Decisions

## Purpose

This document records the architectural choices made for the GreenLeaf Nursery learning project, along with their context, alternatives, and trade-offs. It is intended to make the reasoning reviewable and to provide a reference when the project grows. These records describe the current direction, not immutable rules; decisions can be revisited when requirements or experience change.

## 1. Use a modular monolith

**Context**

The application is initially small. The project is also a guided learning effort spanning backend, frontend, CI/CD, Docker, and cloud deployment. It needs clear responsibility boundaries without unnecessary deployment and operations work.

**Decision**

Build one Spring Boot backend application with logical modules and clear separation of responsibilities. Keep the web client and database as distinct components, but do not split backend capabilities into independently deployed services at the outset.

**Alternatives considered**

- Microservices, with separate independently deployed services.
- A single application with no intentional internal module boundaries.

**Trade-offs**

- A modular monolith reduces the initial number of deployments, network interactions, and operational concerns while preserving a path to clearer internal boundaries.
- Modules share a process and deployment lifecycle, so independent scaling and release are not available per capability.
- If boundaries are ignored, the monolith can become tightly coupled; the decision relies on maintaining separation as features are added.

## 2. Use React and TypeScript for the frontend

**Context**

The application needs a modern browser interface for catalog browsing, search/filter, plant details, and cart capabilities. The project explicitly aims to learn modern frontend development.

**Decision**

Use React with TypeScript, Vite, and Tailwind CSS for the frontend. React provides the UI framework, TypeScript adds static typing to frontend code and API-facing data shapes, Vite provides the development/build tool, and Tailwind CSS provides the styling approach.

**Alternatives considered**

- Other frontend frameworks or libraries.
- JavaScript without TypeScript.
- A server-rendered interface instead of a separate React client.

**Trade-offs**

- This stack supports the stated frontend learning goal and separates browser UI work from backend API work.
- It introduces a separate toolchain and requires care to keep frontend API types aligned with backend contracts.
- TypeScript catches some classes of errors during development, but it does not validate runtime responses or enforce server-side rules.

## 3. Use PostgreSQL for persistence

**Context**

The application includes structured plant, price, stock, category, and care information. The chosen backend persistence approach is Spring Data JPA/Hibernate, with schema evolution managed through Flyway.

**Decision**

Use PostgreSQL as the relational database for application data.

**Alternatives considered**

- Other relational databases.
- A document-oriented or other non-relational database.

**Trade-offs**

- A relational database provides a natural place for structured application records and integrity constraints, and fits the selected JPA/Hibernate and Flyway approach.
- The project must manage database schema changes and relational mapping deliberately.
- Selecting PostgreSQL does not by itself settle schema design, indexing, hosting, backup, or operational policies.

## 4. Model PlantCareRequirements initially as an embedded value object

**Context**

The initial domain has a `Plant` with one set of care requirements: watering interval, sunlight enum, humidity range, watering notes, and care instructions. Seasonal schedules and regional recommendations are possible future extensions, but are not part of the initial model.

**Decision**

Initially model `PlantCareRequirements` as an embedded value object within `Plant`, rather than as a separate entity. The sunlight requirement uses an enum.

**Alternatives considered**

- A separately persisted care-requirements entity associated with each plant.
- A more normalized schedule/region model from the start.

**Trade-offs**

- Embedding reflects the current one-owner, one-set-of-requirements shape and avoids introducing a separate persistence lifecycle before it is needed.
- Updating care fields follows the plant record’s persistence lifecycle; requirements cannot be independently shared or managed without changing the model.
- Seasonal or regional variants may outgrow this shape. Revisit the decision if those capabilities become concrete requirements.

## 5. Use Docker for PostgreSQL

**Context**

The project includes Docker and Docker Compose in its DevOps learning goals and needs a development database for a Spring Boot application.

**Decision**

Run PostgreSQL in Docker for local development, with Docker Compose available to define and start the database consistently. This decision concerns the development database setup; it does not choose a production hosting platform.

**Alternatives considered**

- Install and manage PostgreSQL directly on each developer machine.
- Use a hosted database for development.

**Trade-offs**

- A container provides a repeatable database version and setup across development environments and supports learning Docker/Compose.
- Developers need Docker available, and container data lifecycle and local configuration still need to be managed.
- A local container does not provide production availability, backup, security, or deployment policy; those decisions remain open.

