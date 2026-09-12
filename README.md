# QueryGuard AI

A secure, role-aware AI assistant for enterprise data access.

## Core Idea

Companies store data across dashboards, reports, and databases that take real effort to navigate — different filters, different permissions, different screens just to answer something like "what are my pending tasks?"

QueryGuard AI lets users ask that in plain language instead. The twist is *how* it answers safely: the LLM never touches the database and never generates SQL. It only identifies intent and picks from a fixed set of pre-approved query templates. The backend independently authenticates the user, checks their role, and injects trusted context (user ID, team ID, department ID) before anything is executed.

**Core principle: the AI is not the security layer.** The LLM can help figure out *what* a user wants. Only the backend decides *whether* they're allowed to have it.

## How It Works

1. User logs in → backend issues a JWT with their user ID and role.
2. User asks a question in natural language (e.g. "How is my team performing?").
3. A Supervisor component classifies intent and extracts parameters, calling an LLM API and validating its structured output before trusting any of it.
4. The backend validates that the selected template exists, checks whether the user's role is allowed to use it, and injects trusted identifiers (never taken from user input or LLM output).
5. A parameterized query runs through Spring Data JPA / Hibernate against PostgreSQL.
6. Every request — allowed or denied — gets written to an audit log.

If the LLM tries to select a template that doesn't exist, or one the user's role isn't permitted to use, the backend rejects it before it goes anywhere near the database.

## Functionality

- **Employee** — view own profile, tasks, and performance
- **Manager** — everything an employee can, plus team tasks, team performance, and team metrics
- **Admin** — everything above, plus department metrics, budgets, and audit log access
- Natural language query interface via chat
- Automatic rejection of unsupported or unauthorized queries (no hallucinated answers)
- Full audit trail of every query, its template, and its authorization result

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | React |
| Backend | Java, Spring Boot |
| Persistence | PostgreSQL, Spring Data JPA (Hibernate) |
| Auth | JWT (jjwt) + Spring Security |
| Authorization | Spring Security Role-Based Access Control |
| AI | LLM API (structured/JSON output for intent classification) |
| Build Tool | Maven |
| Deployment | Docker, Docker Compose |
| CI/CD | GitHub Actions |

## Vision

Version 1 is the core SQL-template path described above — natural language in, safe structured queries out, nothing the LLM says trusted without a backend check.

Version 2 extends this with a document-retrieval agent (RAG over a vector DB) so users can also ask unstructured questions like "what's the company's leave policy?" — the same principle carried forward: the AI decides *what* the user is asking, the backend still decides *what they're allowed to see*.
