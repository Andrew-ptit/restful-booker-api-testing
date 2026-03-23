# Restful Booker — API Automation Testing

[![API Tests - Newman](https://github.com/Andrew-ptit/restful-booker-api-testing/actions/workflows/api-tests.yml/badge.svg)](https://github.com/Andrew-ptit/restful-booker-api-testing/actions/workflows/api-tests.yml)

## Overview

Automated API test suite for [Restful-Booker](https://restful-booker.herokuapp.com), a hotel booking system.
Tests run automatically on every code push via GitHub Actions and produce an HTML report as a CI artifact.

---

## What This Project Does

This project automates the full API testing lifecycle for a booking system — from authentication through creating, reading, updating, and deleting bookings. Each request includes assertions that verify response status, data structure, and business logic. A CI pipeline runs the entire suite on every commit and flags failures before they reach production.

---

## Test Report

![Test Report](docs/report.png)

---

## Test Coverage

| Endpoint | Method | What is verified |
| ---------- | -------- | ----------------- |
| `/auth` | POST | Token is returned and valid — wrong credentials are rejected |
| `/booking` | GET | Returns a list of booking IDs |
| `/booking/:id` | GET | All required fields are present in the response |
| `/booking` | POST | Booking is created, ID is returned, schema is correct |
| `/booking/:id` | PUT | All fields are updated and persisted |
| `/booking/:id` | PATCH | Only the targeted field is changed |
| `/booking/:id` | DELETE | Booking is removed and no longer retrievable |
| `/booking/999999` | GET | Non-existent resource returns 404 |

8 requests — 20+ assertions — full CRUD lifecycle in a single run.

---

## Implementation Highlights

**Request Chaining**
The `bookingId` returned from `POST /booking` is stored in a collection variable using `pm.collectionVariables.set()` and automatically injected into all subsequent requests (GET, PUT, DELETE) via `{{bookingId}}` in the URL — no manual ID management needed.

**Authentication Flow**
The token from `POST /auth` is saved as a collection variable and sent in the `Cookie` header for all write operations. The suite also verifies that wrong credentials return a meaningful error (not just a 200 status code).

**Environment Decoupling**
Two environment files (`booker-local.postman_environment.json` and `booker-heroku.postman_environment.json`) store only environment-specific values (`baseUrl`, credentials). The same collection runs against a local Docker server or the Heroku cloud service without any modification.

**CI Artifact**
Each GitHub Actions run uploads the HTML report as a downloadable artifact (retained 30 days), making test results accessible even for failed runs. The workflow is triggered on push, pull request, daily schedule, and manually.

---

## How to Run Locally

1. Start the API server: `docker compose up -d`
2. Install dependencies: `npm install`
3. Run tests: `npm test` (terminal only) or `npm run test:report` (terminal + HTML report)

---

## CI/CD Pipeline

The suite runs automatically on every push to `main` and `develop`, on pull requests, and on a daily schedule at 06:00 UTC. If any assertion fails, the pipeline stops and marks the commit as failed. The HTML report is uploaded as a downloadable artifact after every run.

---

## Project Structure

```text
restful-booker-api-testing/
├── .github/workflows/api-tests.yml
├── collections/restful-booker.postman_collection.json
├── environments/
│   ├── booker-local.postman_environment.json
│   └── booker-heroku.postman_environment.json
├── docs/report.png
├── reports/
├── docker-compose.yml
└── package.json
```

---

## Tech Stack

| Tool | Purpose |
| ---- | ------- |
| Postman | Authoring test collections and writing assertion scripts |
| Newman | Running collections from the command line and in CI |
| newman-reporter-htmlextra | Generating the HTML test report |
| GitHub Actions | Automating test execution on every push |
| Docker | Running the API server locally |
