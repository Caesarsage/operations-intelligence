# Ops Intelligence Platform Specs

This folder contains the product and technical specification for a multi-tenant, BYOK Ops Intelligence Platform focused on connector-driven analysis (Gmail first, then Outlook/IMAP, Slack, Zendesk, Jira), filterable ingestion, and Google Sheets publishing.

## Files

- `roadmap.md`: v1/v2/v3 product roadmap and execution order.
- `architecture.md`: multi-tenant system architecture, data model, RBAC, isolation strategy, and baseline stack.
- `backend-mvp-go.md`: backend MVP build spec for Go with layered architecture (`repository + interface + service + route`) and component specs for Postgres, Redis, RabbitMQ, S3, and Vector.
# operations-intelligence
