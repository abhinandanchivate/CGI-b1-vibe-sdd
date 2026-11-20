# Case Study: Online Book Store — Scope & Solution

Nov 20, 2026 · @Abhi

## Business context and problem statement

PageTurn Books, a fictional mid-size bookseller, needs an online store that lets customers find books and place orders without ever being sold a book that is out of stock, delivered in five phases with Spec-Driven Development (SDD) so every feature is traceable from requirement to test.

Today PageTurn sells only in shops and by phone. Stock lives in a spreadsheet, so customers are promised books that are already sold, and staff re-key orders by hand. An earlier prototype was "vibe coded" with an AI assistant: it shipped quickly, but invented fields, returned every row without pagination, stored a database password in code, had no tests, and checked stock in application code so two buyers could take the last copy. Nobody could say which requirement any piece of code satisfied.

The case study asks teams to rebuild the store the disciplined way with GitHub Spec Kit and Gemini: a project constitution, approved specifications with measurable acceptance criteria, contract-first APIs, a layered FastAPI + PostgreSQL backend, a React or Angular frontend, automated tests at every level, containers and a CI gate. The same case runs through all five phases and grows in two increments — Book Catalog (Phases 2–3) and Cart/Orders (Phase 4) — integrated and assessed in Phase 5.

| Prototype failure | How this solution prevents it |
| --- | --- |
| Invented fields | Approved spec + contract; input schemas reject unknown fields; contract drift test in CI |
| No pagination | Every list paginated, default 20, max 50, enforced on input |
| Password in code | Environment-only configuration; secret scanning before commit and in CI |
| No tests | Test-first tasks; every acceptance criterion has an automated test |
| Overselling the last copy | Conditional stock update inside one transaction, proven by a repeated concurrency test |
| No traceability | IDs on every requirement, criterion, task, commit and test; matrix generated automatically |

### Business goals and success measures

PageTurn's leadership wants three outcomes from the first release. Customers must be able to find a book and buy it online without phoning the shop. The store must never promise a copy that does not exist, because every oversold order today costs a refund, an apology call and often the customer. And staff must stop re-keying orders, so that the time they spend today on data entry moves to customer service.

| Goal | Success measure for the training release | How it is demonstrated |
| --- | --- | --- |
| Customers self-serve | A customer goes from search to placed order in under 1 minute | Playwright purchase journey; live demo |
| No overselling | Zero negative stock and zero double-sold last copies across 25 concurrent runs | Concurrency test suite in CI |
| No re-keying | Every order is stored with its lines, prices and status | Order view (FR-010) and database records |
| Trustworthy delivery | Every line of code traces to an approved requirement and a passing test | Generated traceability matrix at 100% |

### Current state and target state

| Area | Today | Target |
| --- | --- | --- |
| Sales channel | Shop counter and phone | Web store plus existing channels |
| Stock | Spreadsheet updated by hand, often late | Database is the single source of truth; stock changes only inside checkout or by an administrator |
| Orders | Written down, then typed into the spreadsheet | Created automatically at checkout with a status |
| Prices | Changed in the spreadsheet; history lost | Changed by an administrator; each order keeps the price the customer paid |
| Software delivery | AI-generated prototype with no specification or tests | Specification-first, test-first, reviewed and gated by CI |

### Why the prototype failed, and what the training teaches

The prototype was not slow to build; it was impossible to trust. The AI assistant filled every gap in the prompt with its own guesses, so the code contained fields nobody asked for and missed rules nobody wrote down. Without tests, nobody noticed that two customers could buy the last copy at the same moment. Without IDs, nobody could check whether a requirement had been implemented at all.

Spec-Driven Development fixes this by changing what the AI is given. Instead of a vague prompt, the AI works from an approved specification, a technical plan and a numbered task list, all governed by a project constitution. Humans approve each artifact before the next one is generated, and automated gates check the result. The AI still writes most of the code, but it can no longer decide what the code should do.

## Development approach: Spec-Driven Development with Spec Kit

Every feature passes through the same governed sequence: a constitution sets the rules once, then each feature moves from specification to plan to tasks to code, with a human approval or an automated check between every step.

### The workflow, step by step

1. **Constitution** (`/speckit.constitution`) — written once per project. It records the non-negotiable rules (M1–M6) and the standards (S1–S8) that every later artifact must obey. It changes only through a pull request approved by the architect, with a version bump.
2. **Specify** (`/speckit.specify`) — describes *what* the feature does and *why*, in business language: user stories, acceptance criteria, edge cases, out-of-scope items. It contains no frameworks or tables. Spec Kit creates a numbered branch and folder for the feature.
3. **Clarify** (`/speckit.clarify`) — the AI asks targeted questions about anything ambiguous, and the product owner's answers are written into the spec with a date. This is where business rules such as "PUT sets the quantity, it does not add" are settled.
4. **Checklist** (`/speckit.checklist`) — tests the requirements themselves: is every acceptance criterion measurable, does every list state its pagination, is ownership stated for every per-customer resource.
5. **Gate — product owner approval.** The spec moves to Approved. Nothing technical starts before this.
6. **Plan** (`/speckit.plan`) — describes *how*: stack, layers, data model, API contract, design decisions with rejected alternatives, and a Constitution Check that must pass.
7. **Gate — architect approval** of the plan, data model and contract.
8. **Tasks** (`/speckit.tasks`) — a numbered, test-first task list. For every user story, the failing tests come before the implementation tasks that make them pass.
9. **Analyze** (`/speckit.analyze`) — a read-only cross-check that every criterion has a task, every task has a requirement, and nothing conflicts with the constitution. Critical findings block implementation.
10. **Implement** (`/speckit.implement`) — the AI executes one phase of tasks at a time; a human reviews each diff before it is committed.
11. **Converge and verify** — confirm that nothing in the spec is left unimplemented, regenerate traceability, and mark the spec Verified once CI is green.

### Artifacts each feature produces

| Artifact | Answers | Approved by |
| --- | --- | --- |
| spec.md | What and why: stories, acceptance criteria, edge cases, scope | Product owner |
| checklists/requirements.md | Are the requirements complete and measurable? | Product owner |
| plan.md | How: architecture, Constitution Check, decisions | Architect |
| research.md | Why each technical choice, and what was rejected | Architect |
| data-model.md | Tables, columns, constraints, indexes | Architect |
| contracts/openapi.yaml | Exact API: paths, statuses, schemas, errors | Architect |
| quickstart.md | How to verify each criterion by hand | Team |
| tasks.md | Ordered, test-first work with IDs | Team, checked by analyze |

### The .specify folder

The `.specify/` folder is committed with the code, because it is the project's rulebook: every Spec Kit command reads it, and every team member and AI session must see the same version.

```text
.specify/
├── memory/constitution.md        # project law: M1–M6, S1–S8
├── scripts/bash/                 # Spec Kit helpers: create feature branches, check prerequisites
└── templates/
    ├── spec-template.md …        # Spec Kit core templates — never edited
    └── overrides/                # PageTurn's own templates — take precedence over core
        ├── spec-template.md      #   FR/AC IDs, error codes, NFRs touched
        ├── plan-template.md      #   M1–M6 Constitution Check, decision table
        ├── tasks-template.md     #   global task numbers, test-first ordering
        └── checklist-template.md #   PageTurn requirement-quality checks
```

Customising through `overrides/` rather than editing the core templates means a Spec Kit upgrade never wipes out PageTurn's conventions. The constitution and overrides are protected by Code Owners, and the local guard blocks accidental edits to Spec Kit's own files.

### The constitution in brief

| Rule | Statement | Consequence of violation |
| --- | --- | --- |
| M1 Spec-first | No code without an approved spec, plan and task; every commit names its task | Blocker |
| M2 No secrets | Configuration only from the environment; secrets never in code, prompts, logs or Git | Blocker |
| M3 Test-first, traced | Each criterion has a test committed failing before the code; traceability 100% and generated | Blocker |
| M4 Layering | Router → service → repository → model; rules only in services | Blocker |
| M5 Contract-first | Versioned REST; problem-JSON errors with catalogue codes; unknown input fields rejected | Blocker |
| M6 Integrity | Exact money; stock never negative; multi-row changes in one transaction, locks in a fixed order | Blocker |
| S1–S8 | Pagination, p95 performance, one-command startup, human approval, observability, accessibility, supply-chain hygiene, AI-governance evidence | Major unless justified in the plan |

### Identifiers that make traceability possible

Requirements are FR-0NN, acceptance criteria AC-\<FR>.\<n>, non-functional requirements NFR-0N, tasks T0NN in one sequence across all features, and decisions D\<n>. Commits read `<type>(<scope>): <summary> [T0NN][AC-x.y]`, and every test's docstring starts with the criteria it proves. A script joins these IDs to produce the traceability matrix, so the evidence can be regenerated by anyone at any time.

## Scope

The scope is two vertical slices (Catalog, Cart/Orders) plus the engineering controls that make them trustworthy; payments, real authentication and production hosting are out.

| In scope | Out of scope (extensions) |
| --- | --- |
| Browse with pagination, filter by author's full name, search titles (partial, case-insensitive), view book detail | Real payment gateway (checkout records an order only) |
| Admin create book and update price/stock (role simulated with headers and an environment token) | Identity provider, SSO, OAuth; deleting books |
| Server-side cart: set quantity (1–99), remove, summary with totals | Reviews, ratings, recommendations, wish lists |
| Checkout: validate cart, create order, copy prices, decrement stock atomically | Shipping, tax, invoices, email, order cancellation |
| View own order and its status (PLACED) | Idempotency keys; price-change confirmation at checkout |
| PostgreSQL with migrations and synthetic seed data | Production cloud deployment and scaling |
| pytest, HTTPX, Schemathesis, Playwright + axe; Docker Compose; GitHub Actions CI; PR review | Load testing beyond the p95 smoke; building both React and Angular |

**Stakeholders and roles**

| Role | Interest | Played by | Approves |
| --- | --- | --- | --- |
| Customer | Find books, buy without stock surprises | Participants as testers | — |
| Store administrator | Keep catalog and prices correct | Participants as testers | — |
| Product owner | Value and acceptance | Business product lead | Specs, clarifications |
| Architect / reviewer | Rules and design quality | Lead architect + peer team | Constitution, plans, contracts, PRs |
| Developers + AI agents | Deliver tasks | Teams with Gemini | Nothing alone: AI drafts, humans approve |

**Non-functional requirements**

| ID | Requirement | Measure | Verified by |
| --- | --- | --- | --- |
| NFR-01 Performance | Catalog list and detail respond quickly on seed data | p95 < 300 ms locally | Committed script: 20 warm-up + 200 sequential requests per endpoint on the compose stack |
| NFR-02 Scalability of lists | Lists are always paginated | Default 20, max 50, enforced on input | AC-1.2, AC-1.3 |
| NFR-03 Data integrity | Stock never negative; order + stock change in one transaction | Concurrency tests pass 25 of 25 runs | AC-9.2, AC-9.4, AC-9.6; DB CHECK constraints |
| NFR-04 Security | No secrets in code, prompts or Git; inputs validated; customers see only their own data | Secret scan + 400/401/403/422 tests pass | gitleaks pre-commit and CI; validation and ownership ACs |
| NFR-05 Quality | Every AC has an automated test | Generated traceability matrix 100% | `make trace` |
| NFR-06 Operability | Whole stack starts with one command; liveness and readiness endpoints | `docker compose up` + GET /health and /ready = 200 | Compose healthchecks; CI e2e job |

**Assumptions and constraints:** one frontend track per cohort; synthetic data only; single currency INR with 2 decimals; identity is simulated (customers send `X-Customer-Id`; admins send `X-Role: admin` and `X-Admin-Token`), and the application refuses to start with simulated identity in a production environment; Gemini Code Assist enterprise seats, Spec Kit and tool versions are pinned by the organisation and validated before delivery; Phase 5 is protected for integration and assessment.

### Why the scope is drawn this way

The two slices were chosen because together they contain every hard problem a real store has, at a size a team can finish within the five phases. The catalog teaches validation, pagination, search and role checks. Cart and checkout teach money arithmetic, ownership and, above all, concurrency — the one rule that cannot be checked by reading the code and must be proven by a test.

Everything left out is either a separate discipline or would hide the lesson. A payment gateway adds vendor sandboxes and webhooks but no new SDD practice. A real identity provider would consume significant effort on configuration, so identity is simulated with headers and an environment token, and the application refuses to start in that mode in a production environment so the shortcut cannot leak. Cancellation is deliberately left for a later increment; the order status column already allows CANCELLED so it can be added without a migration. Idempotency keys and price-change confirmation at checkout are noted as extensions because the rules that make the first release safe — checkout uses the price at the moment of purchase, and the UI blocks double submission — are already in the specification.

### Key user journeys

| Journey | Actor | Steps | Requirements |
| --- | --- | --- | --- |
| Find a book | Customer | Open the store → type part of a title or enter an author's full name → page through results → open a book to see price and stock | FR-001–004 |
| Keep the catalog right | Administrator | Enter the admin token → add a new book, or correct a price or stock figure → the change is visible to customers immediately | FR-005, FR-006 |
| Build a cart | Customer | Add books from the detail page → adjust quantities or remove lines → see line totals and the grand total | FR-007, FR-008 |
| Buy | Customer | Place the order → stock is reserved and the cart emptied in one step, or the customer is told which book sold out | FR-009 |
| Check an order | Customer | Open the order page → see lines, prices paid, total and status PLACED | FR-010 |

### How to read the non-functional requirements

Each NFR is written so it can fail. "Fast" means p95 under 300 ms, measured by a committed script after a warm-up, not by impression. "Paginated" means the request is rejected if it asks for more than 50 rows, not just that the default is small. "Data integrity" is proven by running the last-copy race 25 times in a row, because a race that passes once proves very little. "Security" includes ownership: a customer asking for someone else's order gets a 404, so order numbers cannot be used to probe the system. "Operability" separates liveness (the process is up) from readiness (the database answers), so the stack only accepts traffic when it can actually serve it.

## Requirements and acceptance criteria

Ten functional requirements cover both increments with 38 Given/When/Then acceptance criteria; every criterion names the HTTP status, the problem `code` and the resulting state, and has one automated test in the reference solution.

| ID | Increment | Requirement | Acceptance criteria |
| --- | --- | --- | --- |
| FR-001 | Catalog | List books, paginated with total count | AC-1.1–1.3 |
| FR-002 | Catalog | Filter by author's full name, case-insensitive | AC-2.1, 2.2 |
| FR-003 | Catalog | Search titles: partial, case-insensitive, wildcards literal, sorted by title | AC-3.1–3.4 |
| FR-004 | Catalog | View book detail | AC-4.1, 4.2 |
| FR-005 | Catalog | Admin creates a book | AC-5.1–5.6 |
| FR-006 | Catalog | Admin updates price and/or stock | AC-6.1–6.5 |
| FR-007 | Orders | Set cart quantity, remove from cart | AC-7.1–7.6 |
| FR-008 | Orders | Cart summary with line and grand totals | AC-8.1, 8.2 |
| FR-009 | Orders | Checkout creates order atomically without overselling | AC-9.1–9.6 |
| FR-010 | Orders | View own order and status | AC-10.1, 10.2 |

### User stories

| Story | Priority | As a… I want… so that… | Independent test |
| --- | --- | --- | --- |
| US1 Browse and find | P1 | As a customer I want to search, filter and page through books so that I can find one without calling the shop | Searching "clean" on seed data shows exactly two titles, and opening "Clean Code" shows its price and stock |
| US2 Maintain catalog | P2 | As an administrator I want to add books and correct price and stock so that customers always see accurate information | Changing a price is visible on the next customer read |
| US3 Build a cart | P1 | As a customer I want to collect books with quantities and see the cost so that I know what I will pay | Adding 2 × Clean Code shows a line total of 998.00 |
| US4 Check out safely | P1 | As a customer I want to place an order that is either fully accepted or clearly refused so that I am never sold a book that is gone | The last-copy race produces exactly one order |
| US5 See my order | P2 | As a customer I want to view my order and its status so that I know what I bought and what I paid | The order page shows lines, prices paid and PLACED |

P1 stories form the minimum sellable store; P2 stories complete it. Each story can be demonstrated on its own, which lets a team that falls behind still show working value at a gate.

### Business rules

| Rule | Statement | Where it is enforced |
| --- | --- | --- |
| BR-1 | A book is identified for uniqueness by title and author together, ignoring capitalisation | Database unique index; AC-5.3 |
| BR-2 | The author filter matches a full name; partial text search applies to titles only | Repository query; AC-2.1–3.3 |
| BR-3 | Search text is matched literally; characters such as % and \_ have no special meaning | Escaped query; AC-3.2 |
| BR-4 | Only price and stock can be changed after a book is created | Update schema; AC-6.5 |
| BR-5 | A cart holds one line per book with a quantity from 1 to 99; setting a quantity replaces it | Cart schema and upsert; AC-7.3, 7.5 |
| BR-6 | The cart's stock check is a courtesy; only checkout guarantees availability | Checkout transaction; AC-9.2, 9.4 |
| BR-7 | An order is all-or-nothing: every line is reserved or none is | Single transaction; AC-9.4 |
| BR-8 | An order records the price at the moment of checkout and never changes afterwards | unit\_price copied; AC-9.5 |
| BR-9 | Totals are computed per line, rounded half-up to 2 decimals, then summed | Money helpers; AC-8.1 |
| BR-10 | A customer can see only their own cart and orders; others' orders do not exist for them | Scoped queries; AC-10.2 |

### How the acceptance criteria are written

Every criterion follows Given/When/Then and states three things: the HTTP status, the problem `code` for errors, and the state afterwards ("stock is 3 and the cart is empty"), because a correct response with a wrong database state is still a failure. Criteria use concrete seed values rather than "some book", so a test can be written from them without interpretation. Where a rule could be read two ways — such as whether "Martin" should match "Martin Fowler" — the criterion states the chosen behaviour explicitly (AC-2.2), so no team builds something different.

Seed data holds 45 synthetic books, including "Clean Code" and "Clean Architecture" by Robert C. Martin and "Refactoring" by Martin Fowler, and no other title containing "clean".

**Catalog**

- **AC-1.1** Given 45 books, When GET /api/v1/books?page=1&size=20, Then 200 with 20 items, total=45, page=1, size=20.
- **AC-1.2** Given 45 books, When no paging is given, Then 20 items and size=20.
- **AC-1.3** Given any data, When size=51, size=0 or page=0, Then 400 INVALID\_PAGINATION.
- **AC-2.1** Given books by "Robert C. Martin" and "Martin Fowler", When author=robert c. martin, Then only Robert C. Martin's books.
- **AC-2.2** Given the same data, When author=Martin, Then 200 with no items (the filter matches full names only).
- **AC-3.1** Given "Clean Code", "Clean Architecture", "Refactoring", When q=CLEAN, Then exactly "Clean Architecture", "Clean Code" in that order.
- **AC-3.2** Given no title contains "%", When q=%, Then 200 with no items.
- **AC-3.3** Given the seed, When q=clean and author=Martin Fowler, Then no items (filters combine with AND).
- **AC-3.4** Given any data, When q exceeds 200 characters, Then 400 INVALID\_QUERY.
- **AC-4.1** Given a book exists, When GET /api/v1/books/{id}, Then 200 with id, title, author, price ("499.00") and stock.
- **AC-4.2** Given no book 9999, When GET /api/v1/books/9999, Then 404 BOOK\_NOT\_FOUND.
- **AC-5.1** Given an admin, When POST a valid book, Then 201 with a Location header and a body equal to the input plus id.
- **AC-5.2** Given an admin, When price is 0 or negative, Then 422 VALIDATION\_ERROR.
- **AC-5.3** Given "Clean Code" by Robert C. Martin exists, When POST "clean code" by "robert c. martin", Then 409 BOOK\_DUPLICATE.
- **AC-5.4** Given a non-admin or a wrong token, When POST, Then 403 FORBIDDEN and nothing is stored.
- **AC-5.5** Given an admin, When the body has an unknown field such as isbn13, Then 422 VALIDATION\_ERROR.
- **AC-5.6** Given an admin, When price has 3 decimals, Then 422 VALIDATION\_ERROR.
- **AC-6.1** Given an admin, When PATCH stock=-1, Then 422 VALIDATION\_ERROR and stock is unchanged.
- **AC-6.2** Given an admin PATCHes price to 499.00, When the book is read, Then price is "499.00".
- **AC-6.3** Given an admin, When PATCH an unknown book, Then 404 BOOK\_NOT\_FOUND.
- **AC-6.4** Given a non-admin, When PATCH, Then 403 FORBIDDEN.
- **AC-6.5** Given an admin, When the body is empty, contains null, or contains title, Then 422 VALIDATION\_ERROR.

**Cart and orders**

- **AC-7.1** Given stock=5, When PUT /api/v1/cart/items/{id} quantity=2, Then 200 and the cart holds that book × 2.
- **AC-7.2** Given stock=3, When PUT quantity=4, Then 409 OUT\_OF\_STOCK with book\_id and available=3.
- **AC-7.3** Given the cart holds 2, When PUT quantity=1, Then the cart holds 1 (PUT sets, it does not add).
- **AC-7.4** Given the cart holds the book, When DELETE it, Then 200 without it; DELETE again → 404 CART\_ITEM\_NOT\_FOUND.
- **AC-7.5** Given any book, When quantity is 0 or 100, Then 422 VALIDATION\_ERROR; When the book does not exist, Then 404 BOOK\_NOT\_FOUND.
- **AC-7.6** Given no X-Customer-Id header, When any cart or order call is made, Then 401 UNAUTHENTICATED.
- **AC-8.1** Given 2 × a 499.00 book and 1 × a 350.50 book, When GET /api/v1/cart, Then line totals "998.00" and "350.50" and total "1348.50".
- **AC-8.2** Given a new customer, When GET /api/v1/cart, Then 200 with no items and total "0.00".

```gherkin
AC-9.1 Checkout succeeds
  Given a cart with 2 x "Clean Code" and stock = 5
  When the customer POSTs /api/v1/orders
  Then the response is 201 with status PLACED and total = 2 x price
  And stock for "Clean Code" is 3 and the cart is empty

AC-9.2 Last copy, two buyers
  Given stock = 1 and two carts each holding 1 copy
  When both customers check out at the same time
  Then exactly one order is created (201)
  And the other receives 409 OUT_OF_STOCK and stock is 0, never -1
```

- **AC-9.3** Given an empty cart, When POST /api/v1/orders, Then 422 CART\_EMPTY.
- **AC-9.4** Given a cart with book A (enough stock) and book B (not enough), When checkout, Then 409 OUT\_OF\_STOCK naming B; A's stock, the cart and the order count are unchanged.
- **AC-9.5** Given an order placed at 499.00, When the admin changes the price to 550.00, Then the order still shows unit\_price "499.00".
- **AC-9.6** Given two carts each holding books A and B with enough stock, When both check out at the same time, Then both get 201 and no request returns 500.
- **AC-10.1** Given the customer's order, When GET /api/v1/orders/{id}, Then 200 with items (book\_id, title, quantity, unit\_price, line\_total), total and status PLACED.
- **AC-10.2** Given another customer's order or no such order, When GET /api/v1/orders/{id}, Then 404 ORDER\_NOT\_FOUND.

Business rules that shape these criteria: the cart's stock check is a courtesy, and checkout is the only guarantee; an order uses the price at the moment of checkout and shows it; the UI disables "Place order" while a checkout is in flight.

**Error codes** (one catalogue shared by backend, frontend and tests)

| Code | Status | Raised when |
| --- | --- | --- |
| INVALID\_PAGINATION | 400 | page < 1 or size outside 1–50 |
| INVALID\_QUERY | 400 | q > 200 or author > 120 characters |
| UNAUTHENTICATED | 401 | Customer header missing on cart/order calls |
| FORBIDDEN | 403 | Admin route without valid admin headers |
| BOOK\_NOT\_FOUND | 404 | Unknown book |
| CART\_ITEM\_NOT\_FOUND | 404 | Removing a book that is not in the cart |
| ORDER\_NOT\_FOUND | 404 | Unknown order or another customer's order |
| BOOK\_DUPLICATE | 409 | Same title and author, ignoring case |
| OUT\_OF\_STOCK | 409 | Quantity exceeds stock; body carries book\_id and available |
| CHECKOUT\_CONFLICT | 409 | Database deadlock or serialization failure; client may retry |
| VALIDATION\_ERROR | 422 | Body fails validation, including unknown fields |
| CART\_EMPTY | 422 | Checkout with no items |
| INTERNAL\_ERROR | 500 | Unexpected error; generic detail plus request\_id |

## Solution architecture

The runtime is five layers — frontend, API routers, services, repositories and PostgreSQL — with every business rule in the service layer; Gemini and Spec Kit shape how code is produced but never run in production.

&#91;embedded content: reference architecture · 5 runtime layers, SDD toolchain on top, cross-cutting concerns beside\]

Routers never touch the database, so every rule is testable without HTTP; the cross-cutting concerns apply to every layer and are enforced by tests and CI rather than by convention.

### Layer responsibilities

| Layer | Owns | Must never | Tested by |
| --- | --- | --- | --- |
| Frontend | Screens, URL state, formatting INR, mapping error codes to messages, disabling buttons while pending | Calculate money; call fetch outside the typed client; show raw error detail | Playwright + axe |
| API routers | HTTP shape: paths, status codes, headers such as Location, request validation, wiring dependencies | Import models, repositories or SQLAlchemy; contain business rules | API tests; architecture test |
| Services | Business rules, transactions (one per use case), lock ordering, raising domain errors | Return ORM models to routers; know about HTTP | Unit and API tests; concurrency tests |
| Repositories | Queries, the conditional stock update, upserts, ownership-scoped reads | Commit, begin or roll back; decide business outcomes | API and concurrency tests; architecture test |
| Database | Durable state and the last line of defence: CHECK, UNIQUE, FK constraints | Hold business logic in triggers | Migration round trip; constraint tests |

### Technology stack

| Concern | Choice |
| --- | --- |
| Backend | Python 3.12, FastAPI, SQLAlchemy 2 (psycopg 3), pydantic v2, pydantic-settings |
| Database | PostgreSQL 16 with pg\_trgm; Alembic migrations; idempotent synthetic seed command |
| Frontend | React 18 + TypeScript + Vite (Angular track applies the same rules) |
| Tests | pytest, HTTPX, Schemathesis, Playwright, axe |
| Delivery | Docker Compose (db, api, web), GitHub Actions, pre-commit with gitleaks |
| AI and SDD | Gemini Code Assist (enterprise), GitHub Spec Kit at the organisation's pinned version |

### How a checkout request flows

1. The customer clicks **Place order**; the button disables itself and the typed client sends `POST /api/v1/orders` with the `X-Customer-Id` header.
2. nginx forwards the call to the API and adds a request ID; middleware records it for every log line.
3. The router checks the customer header (401 if missing) and hands the customer ID to the order service. It does nothing else.
4. The service opens one transaction and reads the cart. An empty cart ends here with 422 CART\_EMPTY.
5. For each cart line, in ascending book order, the repository runs one conditional update that subtracts the quantity only if enough stock remains, and returns the price at that moment. If no row is updated, the service raises 409 OUT\_OF\_STOCK naming the book and the copies still available, and the transaction rolls back every line already reserved.
6. With every line reserved, the service writes the order and its lines with the prices just returned, empties the cart, and commits.
7. The router returns 201 with a Location header; the browser navigates to the order page, which shows status PLACED.

### Runtime deployment

`docker compose up` starts three containers: **db** (PostgreSQL, healthy when it accepts connections), **api** (runs migrations, loads seed data, then serves on port 8000; healthy when `/ready` reaches the database) and **web** (nginx on port 5173 serving the built frontend and proxying `/api`). Each container starts only when the one it depends on is healthy, so the same single command works on a laptop, in CI and in the capstone demo.

## API contracts and data model

Nine versioned endpoints under /api/v1 plus two health endpoints serve both increments; all errors use one RFC 9457 problem-JSON shape `{type, title, status, detail, code, instance, request_id}`, and money travels as a 2-decimal string such as "499.00".

| Method | Path | FR | Success | Errors |
| --- | --- | --- | --- | --- |
| GET | /api/v1/books?q=&author=&page=&size= | 001–003 | 200 Page\[BookOut\] | 400 |
| GET | /api/v1/books/{id} | 004 | 200 BookOut | 404 |
| POST | /api/v1/books | 005 | 201 BookOut + Location | 403, 409, 422 |
| PATCH | /api/v1/books/{id} | 006 | 200 BookOut | 403, 404, 422 |
| PUT | /api/v1/cart/items/{book\_id} | 007 | 200 CartOut | 401, 404, 409, 422 |
| DELETE | /api/v1/cart/items/{book\_id} | 007 | 200 CartOut | 401, 404 |
| GET | /api/v1/cart | 008 | 200 CartOut | 401 |
| POST | /api/v1/orders | 009 | 201 OrderOut + Location | 401, 409, 422 |
| GET | /api/v1/orders/{id} | 010 | 200 OrderOut | 401, 404 |
| GET | /health | NFR-06 | 200 (process up) | — |
| GET | /ready | NFR-06 | 200 (database reachable) | 503 |

Query-parameter problems return 400; request-body problems return 422. Admin routes check the role before validating the body, so a non-admin always gets 403.

**Data model** (PostgreSQL 16, managed by Alembic; constraints named so violations map to problem codes)

| Table | Columns | Constraints and indexes |
| --- | --- | --- |
| book | id PK, title VARCHAR(200), author VARCHAR(120), price NUMERIC(10,2), stock INT, created\_at, updated\_at (timestamptz) | price > 0; stock ≥ 0; UNIQUE (lower(title), lower(author)); trigram GIN index on lower(title) for partial search; index on lower(author) |
| cart\_item | id PK, customer\_id VARCHAR(64), book\_id FK → book, quantity INT | quantity > 0; UNIQUE (customer\_id, book\_id) |
| orders | id PK, customer\_id VARCHAR(64), status VARCHAR(16), total NUMERIC(10,2), created\_at | status IN (PLACED, CANCELLED) — CANCELLED reserved for a later increment; total ≥ 0; index on customer\_id |
| order\_item | id PK, order\_id FK → orders, book\_id FK → book, quantity INT, unit\_price NUMERIC(10,2) | quantity > 0; unit\_price > 0, copied at checkout; index on order\_id |

Pydantic schemas mirror the contract and reject anything not in it:

```python
Money = Annotated[Decimal, Field(gt=0, max_digits=10, decimal_places=2),
                  PlainSerializer(lambda v: f"{v:.2f}", return_type=str, when_used="json")]

class BookCreate(BaseModel):
    model_config = ConfigDict(extra="forbid", str_strip_whitespace=True)
    title: str = Field(min_length=1, max_length=200)
    author: str = Field(min_length=1, max_length=120)
    price: Money
    stock: int = Field(ge=0)

class BookUpdate(BaseModel):                     # price and/or stock only; at least one; no nulls
    model_config = ConfigDict(extra="forbid")
    price: Money | None = None
    stock: int | None = Field(default=None, ge=0)

class CartItemIn(BaseModel):
    model_config = ConfigDict(extra="forbid")
    quantity: int = Field(ge=1, le=99)

class Page(BaseModel, Generic[T]):               # page/size limits are enforced on the request
    items: list[T]
    total: int
    page: int
    size: int
```

### API conventions explained

The contract is written and approved before any endpoint is coded, and a CI test compares it with the API the code actually exposes, so the two cannot drift apart. Paths use plural nouns under `/api/v1`; a breaking change would create `/api/v2`, never silently alter v1. Lists always return the page, its size and the total count, so the UI can draw a pager without a second call, and they are sorted by title then id so paging is stable. Creation returns 201 with a `Location` header pointing to the new resource. Money is a string with exactly two decimals because JSON numbers are floating point and would reintroduce rounding errors in the browser.

### Example exchanges

A page of search results:

```http
GET /api/v1/books?q=clean&page=1&size=20

200 OK
{
  "items": [
    {"id": 2, "title": "Clean Architecture", "author": "Robert C. Martin", "price": "650.00", "stock": 3},
    {"id": 1, "title": "Clean Code", "author": "Robert C. Martin", "price": "499.00", "stock": 5}
  ],
  "total": 2, "page": 1, "size": 20
}
```

A validation error, in the single problem shape every error uses:

```http
POST /api/v1/books        (X-Role: admin, X-Admin-Token: …)
{"title": "X", "author": "Y", "price": "-5.00", "stock": 1, "isbn13": "978…"}

422 Unprocessable Content   Content-Type: application/problem+json
{
  "type": "https://pageturn.example/problems/validation-error",
  "title": "Validation error", "status": 422,
  "detail": "price: Input should be greater than 0",
  "code": "VALIDATION_ERROR", "instance": "/api/v1/books",
  "request_id": "8f2c41d7e0b94a5c",
  "errors": [{"field": "price", "message": "Input should be greater than 0"},
             {"field": "isbn13", "message": "Extra inputs are not permitted"}]
}
```

The losing buyer in the last-copy race:

```http
POST /api/v1/orders       (X-Customer-Id: bob)

409 Conflict
{"type": "https://pageturn.example/problems/out-of-stock", "title": "Out of stock",
 "status": 409, "detail": "Not enough stock", "code": "OUT_OF_STOCK",
 "instance": "/api/v1/orders", "request_id": "a91be0c2…", "book_id": 1, "available": 0}
```

The winning buyer's order:

```http
GET /api/v1/orders/7      (X-Customer-Id: alice)

200 OK
{"id": 7, "status": "PLACED", "total": "998.00", "created_at": "2026-11-19T10:42:11Z",
 "items": [{"book_id": 1, "title": "Clean Code", "quantity": 2,
            "unit_price": "499.00", "line_total": "998.00"}]}
```

### Why the data model looks like this

Constraints live in the database as well as in the application, because the database is the last line of defence if code is ever wrong: a stock value below zero or a price of zero is rejected by PostgreSQL itself. Uniqueness of title and author ignores case so that "clean code" cannot be added beside "Clean Code". Title search uses a trigram index because an ordinary index cannot help a search for text in the middle of a title. `order_item.unit_price` is a copy, not a reference, so changing a book's price later never rewrites what a customer paid. Foreign keys are indexed explicitly because PostgreSQL does not do it automatically, and order lookups by customer or order would otherwise scan whole tables.

## Key design decisions and solution highlights

The hardest requirement is AC-9.2 (two buyers, one copy); it is solved in the service layer with a conditional stock update, processed in a fixed lock order, inside one database transaction.

| Decision | Choice | Why | Rejected alternative |
| --- | --- | --- | --- |
| D1 API style | Contract-first REST /api/v1; committed OpenAPI checked against the generated one in CI | Frontend and tests build from one contract; drift fails the build | Code-first, contract generated afterwards |
| D2 Layering | Router → service → repository → model; services own transactions; repositories never commit | Business rules testable without HTTP; enforced by an architecture test | Routers calling the ORM directly |
| D3 Stock safety | `UPDATE book SET stock = stock - qty WHERE id = :id AND stock >= qty RETURNING price` in one transaction | Under READ COMMITTED the second buyer waits for the row lock, then re-checks the WHERE clause and matches nothing; RETURNING gives the price at the moment of locking | Read-then-write check in Python (race condition) |
| D4 Lock order | Process cart lines in ascending book\_id; deadlock or serialization failure → 409 CHECKOUT\_CONFLICT | Concurrent multi-book checkouts never deadlock into a 500 | Cart insertion order |
| D5 Money | NUMERIC(10,2) + Decimal + JSON string; ROUND\_HALF\_UP; unit price copied to order\_item | No float rounding; history survives price changes | float; price looked up at read time |
| D6 Errors | RFC 9457 problem JSON with stable `code` and `request_id` | UI maps codes to messages; tests assert codes; support correlates logs | Free-text error strings |
| D7 Input safety | `extra="forbid"` on all input schemas; literal-wildcard search; length and range limits | No invented or mass-assigned fields; no unbounded requests | Silently ignoring unknown fields |
| D8 Search | Trigram GIN index on lower(title); case-insensitive unique index on title + author | Partial-match search stays fast; "clean code" cannot duplicate "Clean Code" | B-tree on lower(title), which cannot serve `%term%` |
| D9 Ownership | Every cart and order query scoped by customer\_id; another customer's order returns 404 | Prevents reading or enumerating others' orders | 403 for others' orders |
| D10 Config | Environment variables via pydantic-settings; admin token as a secret; simulated identity blocked in production | No secrets in Git (constitution M2) | Hard-coded connection string |
| D11 Frontend | Typed API client from the contract; money kept as strings; URL holds search state | Compile-time field safety; no float maths; shareable searches | Untyped fetch calls scattered in components |
| D12 Operability | `/health` for liveness, `/ready` for the database; request-ID middleware with JSON logs | Compose starts traffic only when the DB is ready; every error is traceable | One health endpoint; plain-text logs |

Reference checkout service:

```python
class OrderService:
    def __init__(self, session: Session, carts: CartRepository, books: BookRepository,
                 orders: OrderRepository) -> None:
        self.session, self.carts, self.books, self.orders = session, carts, books, orders

    def checkout(self, customer_id: str) -> OrderOut:
        try:
            with self.session.begin():                                    # one transaction
                items = self.carts.list_for(customer_id)
                if not items:
                    raise DomainError(ErrorCode.CART_EMPTY, 422, "Your cart is empty")
                lines = []
                for item in sorted(items, key=lambda i: i.book_id):     # fixed lock order (D4)
                    price = self.books.reserve_stock(item.book_id, item.quantity)  # D3
                    if price is None:                                     # someone bought it first
                        raise DomainError(ErrorCode.OUT_OF_STOCK, 409, "Not enough stock",
                                          book_id=item.book_id,
                                          available=self.books.stock_of(item.book_id))
                    lines.append(OrderItem(book_id=item.book_id, quantity=item.quantity,
                                           unit_price=price))             # price at checkout
                total = grand_total(line_total(x.unit_price, x.quantity) for x in lines)
                order = self.orders.add(Order(customer_id=customer_id, status="PLACED",
                                              total=total, items=lines))
                self.carts.clear(customer_id)
                return self._to_out(order)
        except OperationalError as exc:
            if getattr(exc.orig, "sqlstate", None) in {"40P01", "40001"}:
                raise DomainError(ErrorCode.CHECKOUT_CONFLICT, 409, "Please retry") from exc
            raise


class BookRepository:
    def reserve_stock(self, book_id: int, quantity: int) -> Decimal | None:
        stmt = (update(Book)
                .where(Book.id == book_id, Book.stock >= quantity)
                .values(stock=Book.stock - quantity)
                .returning(Book.price))
        return self.session.execute(stmt).scalar_one_or_none()
```

If any line fails, the whole transaction rolls back: no partial order and no lost stock. The 409 carries `book_id` and `available` so the UI can name the book that sold out.

### Why the checkout cannot oversell

Take AC-9.2: one copy of "Clean Code" left, Alice and Bob both press Place order at the same instant.

| Moment | Alice's transaction | Bob's transaction | Stock |
| --- | --- | --- | --- |
| t1 | Begins; reads her cart | Begins; reads his cart | 1 |
| t2 | `UPDATE … SET stock = stock - 1 WHERE id = 1 AND stock >= 1` matches the row and locks it | Runs the same UPDATE and waits for Alice's lock | 0 (uncommitted) |
| t3 | Writes her order, empties her cart, commits; lock released | — | 0 |
| t4 | — | PostgreSQL re-checks Bob's WHERE clause against the committed row: `0 >= 1` is false, no row matches | 0 |
| t5 | — | Service sees no row returned, raises 409 OUT\_OF\_STOCK, rolls back | 0 |

The database, not the application, decides who wins, and it does so while holding a row lock for only a few milliseconds. The version the prototype used — read the stock in Python, check it, then write — fails because both transactions read `stock = 1` at t1, both pass the check, and both write. A CHECK constraint (`stock >= 0`) sits underneath as a final safety net: even buggy code could not store a negative value.

The fixed lock order matters when carts hold several books. If Alice's cart locks book A then B while Bob's locks B then A, each waits for the other and PostgreSQL must abort one of them. Processing lines in ascending book ID means everyone queues in the same order, so that cycle cannot form; if the database still reports a deadlock or serialization failure, the customer receives 409 CHECKOUT\_CONFLICT with an invitation to retry rather than a 500.

### How money stays exact

Prices are stored as NUMERIC(10,2), handled as Python `Decimal`, and sent as strings. Each line total is the unit price times the quantity, rounded half-up to two decimals; the grand total is the sum of line totals. The browser only formats the strings it receives as Indian rupees and never adds or multiplies them, so the figure a customer sees is exactly the figure stored on the order.

### How decisions are recorded

Each decision above is written in `research.md` with its rejected alternative, so a reviewer can see that the choice was deliberate. When a decision changes — for example if a later increment adds a payment step — the change is made in research.md and the plan first, then flows through tasks to code, exactly like a requirement change.

## Quality, governance and CI solution

No code reaches main unless four independent gates pass: the spec gate, the deterministic local guard, the CI pipeline and a human PR review.

| Test level | Tool | What it proves | Example |
| --- | --- | --- | --- |
| Unit | pytest | Service rules and architecture rules without HTTP/DB | Cart total = Σ price × qty; routers never import the ORM |
| API / integration | pytest + HTTPX + PostgreSQL | Contract, status codes, problem codes, persistence | POST /books price=-5 → 422 VALIDATION\_ERROR |
| Contract | Drift test + Schemathesis | Code matches the committed contract; no unhandled 500s | Generated OpenAPI = committed operations |
| Concurrency | pytest, two threads, barrier, 25 runs | Stock never negative; no deadlock | AC-9.2 last copy; AC-9.6 two multi-book carts |
| UI end-to-end | Playwright + axe | Real journey in a browser; no serious accessibility violations | list → cart → checkout → order status |
| Performance | Committed p95 script | NFR-01 on the compose stack | p95 < 300 ms on list, search, detail |

**Governance controls in the reference repository**

- Constitution v1.1.0: MUST rules M1–M6 (spec-first, no secrets, test-first and traced, layering, contract-first, data integrity) are Blockers in PR review; SHOULD rules S1–S8 cover pagination, performance, operability, human approval, observability, accessibility, supply chain and AI governance.
- `.specify/` is committed; project templates live in `.specify/templates/overrides/` so every spec, plan, task list and checklist follows the same conventions; Code Owners protect the constitution, overrides, shared error codes and contracts.
- GEMINI.md and standards keep the AI inside the agreed layering; `.aiexclude` keeps `.env`, keys and build output away from the model; agent roles (test author, backend implementer, frontend implementer, reviewer) have explicit "never edit" boundaries; `evidence/ai-log.md` records each AI session and the human decision.
- Pre-commit guard runs gitleaks, ruff and the unit/API tests, blocks edits to Spec Kit core files, rejects commit messages without a task ID, and warns when tests and code change in one commit.
- GitHub Actions runs four required jobs on every PR: secret scan of full history; backend (lint, migration round trip, unit/API/contract tests with ≥ 85% service coverage, concurrency suite, pip-audit); frontend (lint, build, tests, npm audit); e2e (compose stack, Schemathesis, p95 script, Playwright + axe). Actions are pinned to commit SHAs with read-only permissions.
- Branch protection requires green CI, one Code Owner approval and rebase-only merges with linear history, so every task commit survives; review findings are labelled Blocker / Major / Minor.
- `evidence/traceability.md` maps every FR → AC → task → commit → test and is generated by `scripts/trace.py`, never hand-edited; the capstone is scored on it (25% of the rubric).

### The four gates, in the order code meets them

| Gate | When | What it catches | Who or what decides |
| --- | --- | --- | --- |
| 1 Spec gate | Before any task exists | Vague, untestable or out-of-scope requirements; designs that break the constitution | Product owner (spec), architect (plan and contract), `/speckit.analyze` |
| 2 Local guard | On every commit | Secrets, lint errors, failing unit/API tests, edits to Spec Kit core files, commits without a task ID | Pre-commit hooks, deterministic, whatever the AI produced |
| 3 CI pipeline | On every pull request | Everything the guard checks plus migrations, contract drift, concurrency, end-to-end journeys, accessibility, dependency vulnerabilities, p95 | GitHub Actions, four required jobs |
| 4 Human review | Before merge | Design quality, rule violations the tools cannot see, tests weakened to pass, invented fields | Code Owner reviewer with labelled findings |

The gates overlap on purpose. The guard can be bypassed on a laptop, so CI repeats its checks; CI cannot judge whether a design is sensible, so a human reviews; and none of the later gates can rescue a wrong requirement, so the spec gate comes first.

### Review severities

| Severity | Meaning | Example | Merge allowed? |
| --- | --- | --- | --- |
| Blocker | Breaks a MUST rule or an acceptance criterion | Router queries the database; stock checked in Python before update | No |
| Major | Breaks a SHOULD rule or weakens evidence | Input schema accepts unknown fields; a test assertion relaxed in the same commit as the fix | Only with written justification |
| Minor | Style or clarity | Button without an accessible name | Yes, fix tracked |

### Definition of Ready and Definition of Done

A feature is **ready** for tasks when its spec is Approved with no open clarification markers, every requirement has measurable criteria with status and error code, every code is in the shared catalogue, the requirements checklist is complete, and the plan's Constitution Check passes with the contract approved.

A task is **done** when its test was committed failing first and now passes, its commit carries the task and criterion IDs, and the guard is green. A feature is **done** when every criterion's test is green in CI, contract drift and Schemathesis are clean, service coverage is at least 85%, axe reports no serious violations, p95 is recorded, traceability is 100%, a Code Owner has approved with no open Blockers, and the spec is marked Verified.

### Keeping the AI honest

The AI drafts almost every artifact, so the controls focus on what it is allowed to change. Each agent role works in its own session with written boundaries: the test author never touches application code, the implementers never touch tests, the reviewer never edits at all. Implementation runs one phase at a time so diffs stay reviewable. Unknown input fields are rejected at runtime and the contract is checked in CI, which turns "the AI invented a field" from a review hope into a failing test. Every session is logged in `evidence/ai-log.md` with the prompt, the output and the human's decision — accepted, changed or rejected — and the reason.

**Traceability matrix** (commits are filled in by the generator)

| Requirement | Acceptance criteria | Task | Test |
| --- | --- | --- | --- |
| FR-001 List | AC-1.1, 1.2, 1.3 | T011 | test\_list\_books\_paginated, test\_default\_page\_size, test\_invalid\_pagination\_returns\_400 |
| FR-002 Author filter | AC-2.1, 2.2 | T013 | test\_filter\_books\_by\_author, test\_author\_partial\_name\_returns\_empty |
| FR-003 Search | AC-3.1–3.4 | T012 | test\_search\_books\_by\_title, test\_search\_matches\_wildcards\_literally, test\_search\_and\_author\_combined, test\_query\_too\_long\_returns\_400 |
| FR-004 Detail | AC-4.1, 4.2 | T014 | test\_get\_book\_detail, test\_get\_unknown\_book\_returns\_404 |
| FR-005 Create | AC-5.2, 5.6 | T015 | test\_create\_book\_rejects\_negative\_price, test\_create\_book\_rejects\_three\_decimals |
| FR-005 Create | AC-5.1, 5.3, 5.4, 5.5 | T016 | test\_create\_book\_returns\_201\_with\_location, test\_create\_duplicate\_book\_returns\_409, test\_create\_book\_non\_admin\_returns\_403, test\_create\_book\_unknown\_field\_returns\_422 |
| FR-006 Update | AC-6.1–6.5 | T017 | test\_update\_rejects\_negative\_stock, test\_price\_update\_visible\_on\_get, test\_update\_unknown\_book\_returns\_404, test\_update\_non\_admin\_returns\_403, test\_update\_empty\_or\_title\_returns\_422 |
| FR-001/003/004 UI | AC-1.1, 3.1, 4.1 | T025 | catalog\_smoke.spec.ts |
| FR-007 Cart | AC-7.2, 7.5 | T027 | test\_add\_item\_exceeding\_stock\_returns\_409, test\_add\_unknown\_book\_returns\_404, test\_invalid\_quantity\_returns\_422 |
| FR-007 Cart | AC-7.1, 7.3, 7.4, 7.6 | T028 | test\_set\_quantity\_in\_cart, test\_put\_sets\_not\_adds, test\_remove\_item\_and\_missing\_returns\_404, test\_missing\_customer\_header\_returns\_401 |
| FR-008 Summary | AC-8.1, 8.2 | T029 | test\_line\_and\_grand\_totals, test\_cart\_summary\_totals, test\_empty\_cart\_summary |
| FR-009 Checkout | AC-9.3, 9.4 | T030 | test\_checkout\_empty\_cart\_returns\_422, test\_checkout\_partial\_stock\_rolls\_back |
| FR-009 Checkout | AC-9.1 | T031 | test\_checkout\_creates\_order\_and\_decrements\_stock |
| FR-009 Checkout | AC-9.2 | T032 | test\_concurrent\_checkout\_last\_copy |
| FR-009 Checkout | AC-9.6 | T033 | test\_concurrent\_checkout\_multi\_book\_no\_deadlock |
| FR-009/010 | AC-9.5, 10.1, 10.2 | T034 | test\_order\_keeps\_unit\_price\_after\_price\_change, test\_get\_own\_order, test\_other\_customers\_order\_returns\_404 |
| FR-010 Order status | AC-10.1 | T036 | e2e\_purchase\_journey.spec.ts |

## Delivery mapping to the five phases

The case study grows one increment per phase and is complete, integrated and assessed by the end of Phase 5; each phase ends at a gate and a tag.

| Phase | Case-study increment | Spec Kit flow | Requirements covered | Artifacts produced | Gate |
| --- | --- | --- | --- | --- | --- |
| Phase 1 — Foundation | Baseline store + failure analysis of the vibe-coded prototype | `specify init`, template overrides | — | environment-checklist, failure-log, GEMINI.md, standards, .aiexclude, .specify overrides, CODEOWNERS | Starters run; Gemini answers from GEMINI.md |
| Phase 2 — Specification | Rules and Catalog specification | constitution → specify → clarify → checklist | FR-001–006, NFR-01–06 | constitution.md v1.1.0, error-codes.md, specs/001-book-catalog/spec.md | Constitution ratified; spec Approved |
| Phase 3 — Catalog build | Catalog designed and built | plan → tasks (T001–T026) → analyze → implement | FR-001–006 | plan, research, data model, contract, tasks, API, UI, tests, perf evidence | Catalog tests, E2E smoke and p95 green |
| Phase 4 — Orders build | Cart/Orders built by agents, containerised | specify → clarify → plan → tasks (T027–T046) → analyze → implement | FR-007–010 | specs/002-cart-orders/, agent roles, /trace command, guard, Dockerfiles, docker-compose.yml | Concurrency 25/25 + `docker compose up` + /ready |
| Phase 5 — Release | Integrated store through CI and PR | specify → plan → tasks (T047–T052) for quality gates | All | ci.yml, branch protection, PR review, test reports, generated traceability | All CI jobs green; traceability 100%; capstone demo scored |

### What happens in each phase

**Phase 1 — Foundation: baseline and failure analysis.** Teams verify their tools against the organisation's pinned versions, run the vibe-coded prototype and record each failure with its root cause and the control that will prevent it. They initialise Spec Kit with Gemini, write GEMINI.md (the AI's working agreement), the coding standards and `.aiexclude`, and create the template overrides in `.specify/`. The phase closes when every laptop runs the starter and Gemini answers questions about the layering and money rules from GEMINI.md rather than guessing.

**Phase 2 — Rules and the catalog specification.** Teams ratify the constitution, create the shared error-code catalogue, and specify the catalog in business language. `/speckit.clarify` settles the ambiguous points — author filter versus title search, 400 versus 422, what an administrator may change — and the checklist tests the spec's quality. The product owner approves the spec, and the architect ratifies the constitution and a first amendment, demonstrating change control.

**Phase 3 — Catalog designed and built.** The plan, research notes, data model and contract are produced and approved before any code. Tasks T001–T026 follow test-first order: setup and foundations, then all catalog acceptance tests committed failing, then the code that turns them green, then the frontend and a Playwright smoke test. The phase closes with the p95 measurement and a clean contract check.

**Phase 4 — Cart and orders by agents.** The second feature goes through the same flow, but implementation is split across role-separated agent sessions: one writes the tests, one implements the backend, one the frontend, and a reviewer checks the diffs. The concurrency suite proves the last-copy rule 25 times in a row. The guard, Dockerfiles and compose file make the whole stack start with one command.

**Phase 5 — Integration and assessment.** Teams add the CI pipeline as a small feature of its own, protect the main branch, run a peer review on the capstone pull request with labelled findings, generate the traceability matrix and give a 15-minute demo: the problem, the rules, one requirement traced end to end, the stack starting live, the guard blocking a secret, the review, and the matrix at 100%.

### Common pitfalls reviewers watch for

| Pitfall | Symptom | Correction |
| --- | --- | --- |
| Technology in the spec | spec.md mentions FastAPI or tables | Move it to plan.md and re-run analyze |
| Implementing everything at once | Huge diffs, invented fields | One phase per implement run, reviewed |
| Tests written after code | No failing test commit before the code commit | Redo the task test-first |
| Stock checked in Python | Last-copy test fails intermittently | Conditional update in one transaction |
| Money as floating point in the UI | Totals off by 0.01 | Keep strings from the API; format only |
| Editing Spec Kit core templates | Conventions lost after an upgrade | Use `.specify/templates/overrides/` |
| Squash merges | Task commits disappear from history | Rebase-only merges |

**Assessment rubric**

| Criterion | Weight |
| --- | --- |
| Traceability FR → AC → task → commit → test | 25% |
| Working software against the acceptance criteria | 20% |
| Specification quality | 15% |
| Test quality, including concurrency and contract tests | 15% |
| Architecture and constitution compliance | 10% |
| CI/CD, security and operability | 10% |
| AI governance evidence and reflection | 5% |

Delivery note: keep a reference tag per phase gate (`gate-1-foundation` … `gate-5-release`) so any team that falls behind can resume from the previous gate without losing the rest of the case study. Run a pre-flight on the real laptops one week earlier (proxy access to package registries, Docker and WSL2, Gemini seats, the pinned Spec Kit version).

## Risks and mitigations

| Risk | Likelihood | Impact | Mitigation |
| --- | --- | --- | --- |
| Corporate proxy blocks package registries or container pulls | High | High | Pre-flight a week ahead; internal mirrors; pre-pulled images |
| Gemini unavailable or out of quota | Medium | High | Teams fill the override templates by hand; reference branches |
| A team falls behind a gate | High | Medium | Resume from the previous `gate-N` branch; pair with a peer reviewer |
| Concurrency test is flaky in the demo | Medium | High | Real PostgreSQL, a start barrier, fixed lock order; 25-run evidence captured beforehand |
| AI weakens tests to make them pass | Medium | High | Role boundaries, guard warning, PR template question, reviewer focus |
| Different Spec Kit versions across laptops | Medium | Medium | Pinned version checked on the environment checklist |
| A secret is committed | Low | High | gitleaks before commit and over full history in CI; rotate on any leak |

## Glossary

| Term | Meaning |
| --- | --- |
| SDD | Spec-Driven Development: specifications, plans and tasks drive the code, not ad-hoc prompts |
| Spec Kit | GitHub's toolkit that provides the `/speckit.*` commands, templates and the `.specify/` folder |
| Constitution | The project's non-negotiable rules, read by every Spec Kit command |
| Acceptance criterion (AC) | A Given/When/Then statement that a test can prove true or false |
| Problem JSON | The standard error body (RFC 9457) with a stable `code` |
| p95 | The response time that 95% of requests beat |
| Traceability | The chain FR → AC → task → commit → test, generated from IDs |
| Gate | A check that must pass before work moves to the next step |
