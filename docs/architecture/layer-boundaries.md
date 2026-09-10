# Application Layer Boundaries

## Purpose

The eyewear platform uses an MVC architecture with explicit Controller,
Service, and Repository layers.

The objective is to keep presentation, business logic, persistence, and
database concerns separated.

## Architecture

```text
VIEW
  ↓
CONTROLLER
  ↓
SERVICE
  ↓
REPOSITORY
  ↓
PRISMA / MODEL
  ↓
POSTGRESQL
```
