# Campus Rides — Agentic AI Workshop

## Project Overview

Campus Rides is a campus-based ride-booking system
designed using a role-based Agentic AI workflow.

The project focuses on real-time ride matching,
concurrency control, reliable payment processing,
and consistent business rule enforcement.

## Key Features

- Driver matching within 3 km of the pickup location.
- First successful driver acceptance wins.
- Prevention of driver double-booking.
- Two-minute ride request expiration.
- Fixed fare calculation: $2 + $1 per kilometer.
- Safe cancellation and payment handling.
- Prevention of duplicate charges.
- Support for intermittent network connectivity.

## Agentic AI Workflow

The project was developed using specialized AI agents
orchestrated through Python and the OpenRouter API.

| Agent | Responsibility | Status |
|---|---|---|
| Researcher | Requirement analysis | Completed |
| Product Manager | Product requirements | Completed |
| Architect | System design and scenario analysis | Completed |
| Reviewer | Requirement and design validation | Completed |
| Developer | Application implementation | Not required |
| Deployer | Application deployment | Not required |

## Workflow

Researcher → Product Manager → Architect → Reviewer
→ AI-assisted manual refinement → Final Deliverables

## Project Deliverables

| Document | Description |
|---|---|
| [PRD.md](PRD.md) | Product Requirements Document |
| [DESIGN.md](DESIGN.md) | Technical architecture, APIs, data model and rule enforcement |
| [ANSWERS.md](ANSWERS.md) | Solutions to three concurrency and idempotency scenarios |

## Tools and Technologies

### Agent Execution

- Python
- OpenRouter API
- OpenAI Python SDK
- PyYAML
- Visual Studio Code
- Markdown

### Proposed System Architecture

- PostgreSQL
- PostGIS
- Redis
- REST APIs
- WebSockets
- Transactional Outbox
- Database Transactions
- Payment Idempotency

## System Requirements

- 300 drivers
- 2,000 ride requests per hour
- Driver location updates every 5 seconds
- Safe handling of concurrent requests
- Reliable operation during network interruptions

## Project Status

The required documentation has been generated,
reviewed, and refined.

This project covers requirement analysis and system
design. Application implementation, deployment,
and runtime testing are outside the assignment scope.

## Author

**Mahrukh Khan**

- Roll Number: BSSE24027
- Program: BS Software Engineering
- Assignment: Agentic AI Workshop
