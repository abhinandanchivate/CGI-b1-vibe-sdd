# Case Study: Online Book Store — Scope & Solution

Nov 16, 2026 · @Abhi

## Business context and problem statement

PageTurn Books, a fictional mid-size bookseller, needs an online store that lets customers find books and place orders, built in 5 training days with Spec-Driven Development so every feature is traceable from requirement to test.

Today PageTurn sells only in shops and by phone. Stock is tracked in a spreadsheet, so customers are promised books that are already sold, and staff re-key orders by hand. An earlier prototype was “vibe coded” with an AI assistant: it shipped quickly, but invented fields, had no pagination, stored a database password in code and had no tests. Nobody could say which requirement any piece of code satisfied.

The case study asks teams to rebuild the store the disciplined way: a project constitution, approved specifications with measurable acceptance criteria, contract-first APIs, a layered FastAPI + PostgreSQL backend, a React or Angular frontend, automated tests, containers and a CI gate. The same case runs through all five days and grows in two increments: Book Catalog (Days 2–3) and Cart/Orders (Day 4), integrated and assessed on Day 5.

## Scope

The scope is two vertical slices (Catalog, Cart/Orders) plus the engineering controls that make them trustworthy; payments, real authentication and production hosting are out.

| In scope | Out of scope (extensions) |
| --- | --- |
| Browse, search (title/author), paginate and view book detail | Real payment gateway (checkout records an order only) |
| Admin create and update book (role simulated with a header or seeded token) | Full identity provider, SSO, OAuth flows |
| Cart: add, change quantity, remove, summary with totals | Reviews, ratings, recommendations, wish lists |
| Checkout: validate cart, create order, decrement stock atomically | Shipping, tax engines, invoices, email notifications |
| View order and its status (PLACED) | Production cloud deployment and scaling |
| PostgreSQL persistence with migrations and seed data | Deep MCP integrations; building both React and Angular |
| Tests (pytest, HTTPX, Playwright), Docker Compose, GitHub Actions CI, PR review | Performance/load testing beyond the p95 smoke target |

Stakeholders and roles:

| Role | Interest | Played by |
| --- | --- | --- |
| Customer | Find books, buy without stock surprises | Participants as testers |
| Store administrator | Keep catalog and prices correct | Participants as testers |
| Product owner | Approves specs and acceptance criteria | Trainer |
| Architect / reviewer | Approves constitution, design and PRs | Trainer + peer team |
| Developers + AI agents | Implement tasks; AI drafts, humans approve | Participant teams with Gemini |

Non-functional requirements:

| ID | Requirement | Measure |
| --- | --- | --- |
| NFR-01 Performance | Catalog list and detail respond quickly on seed data | p95 < 300 ms locally |
| NFR-02 Scalability of lists | Lists are always paginated | page size default 20, max 50 |
| NFR-03 Data integrity | Stock never negative; order + stock change in one transaction | Concurrency test passes |
| NFR-04 Security | No secrets in code, prompts or Git; inputs validated | Secret scan + 422 tests pass |
| NFR-05 Quality | Every AC has an automated test | Traceability matrix 100% |
| NFR-06 Operability | Whole stack starts with one command; health endpoint | `docker compose up` + GET /health = 200 |

Assumptions and constraints: one frontend track per cohort; synthetic data only; single currency (INR, 2 decimals); Gemini Code Assist enterprise seats, Spec Kit and tool versions are fixed by the organization and validated before delivery; Day 5 is protected for integration and assessment.

## Requirements and acceptance criteria

Ten functional requirements cover both increments; each has at least one Given/When/Then acceptance criterion and one automated test in the reference solution.

| ID | Increment | Requirement | Key acceptance criterion |
| --- | --- | --- | --- |
| FR-001 | Catalog | List books, paginated | GET /books?page=1&size=20 → 200, ≤ 20 items, total count |
| FR-002 | Catalog | Filter by author | ?author=Martin returns only that author’s books |
| FR-003 | Catalog | Search by title (case-insensitive, partial) | ?q=clean returns “Clean Code” and “Clean Architecture”, sorted by title |
| FR-004 | Catalog | View book detail | Unknown id → 404 problem JSON |
| FR-005 | Catalog | Admin creates a book | Valid body → 201 + Location; price ≤ 0 → 422; duplicate title+author → 409; non-admin → 403 |
| FR-006 | Catalog | Admin updates price/stock | stock < 0 → 422; change visible on next GET |
| FR-007 | Orders | Add to cart / change quantity / remove | Quantity > stock → 409 with available stock |
| FR-008 | Orders | Cart summary | Line totals and grand total = Σ price × qty, 2 decimals |
| FR-009 | Orders | Checkout creates order | Order PLACED, cart emptied, stock decremented in one transaction |
| FR-010 | Orders | View order status | GET /orders/{id} shows items, total, status |

Critical acceptance criteria written in full:

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

## Solution architecture

&#91;embedded content: reference architecture · 5 layers, SDD toolchain on top, cross-cutting concerns beside\]

Gemini and Spec Kit drive how code is produced but never sit in the runtime path; routers never touch the database directly, so every business rule lives in the testable service layer.

## API contracts and data model

Nine versioned endpoints under /api/v1 serve both increments; all errors use one problem-JSON shape `{type, title, status, detail, code}`.

| Method | Path | FR | Success | Errors |
| --- | --- | --- | --- | --- |
| GET | /books?q=&author=&page=&size= | 001–003 | 200 Page\[BookOut\] | 400 |
| GET | /books/{id} | 004 | 200 BookOut | 404 |
| POST | /books | 005 | 201 BookOut | 403, 409, 422 |
| PATCH | /books/{id} | 006 | 200 BookOut | 403, 404, 422 |
| PUT | /cart/items/{book\_id} | 007 | 200 CartOut | 404, 409, 422 |
| DELETE | /cart/items/{book\_id} | 007 | 200 CartOut | 404 |
| GET | /cart | 008 | 200 CartOut | — |
| POST | /orders | 009 | 201 OrderOut | 409, 422 |
| GET | /orders/{id} | 010 | 200 OrderOut | 404 |

Data model (PostgreSQL, managed by Alembic):

| Table | Columns | Constraints |
| --- | --- | --- |
| book | id PK, title, author, price NUMERIC(10,2), stock INT, created\_at, updated\_at | price > 0; stock ≥ 0; UNIQUE(title, author); index on lower(title) |
| cart\_item | id PK, customer\_id, book\_id FK → book, quantity INT | quantity > 0; UNIQUE(customer\_id, book\_id) |
| orders | id PK, customer\_id, status, total NUMERIC(10,2), created\_at | status IN (PLACED, CANCELLED) |
| order\_item | id PK, order\_id FK → orders, book\_id FK → book, quantity, unit\_price | unit\_price copied at checkout |

Pydantic schemas mirror the contract, for example:

```python
class BookCreate(BaseModel):
    title: str = Field(min_length=1, max_length=200)
    author: str = Field(min_length=1, max_length=120)
    price: Decimal = Field(gt=0, max_digits=10, decimal_places=2)
    stock: int = Field(ge=0)

class Page(BaseModel, Generic[T]):
    items: list[T]
    total: int
    page: int
    size: int = Field(le=50)
```

## Key design decisions and solution highlights

The solution’s hardest requirement is AC-9.2 (two buyers, one copy); it is solved in the service layer with a conditional stock update inside one database transaction.

| Decision | Choice | Why | Rejected alternative |
| --- | --- | --- | --- |
| D1 API style | Contract-first REST, /api/v1, OpenAPI from Pydantic | Frontend and tests build from the same contract | Code-first, contract generated afterwards |
| D2 Layering | Router → service → repository → model | Business rules testable without HTTP or DB | Routers calling the ORM directly |
| D3 Stock safety | Conditional UPDATE … WHERE stock ≥ qty in one transaction | Correct under concurrency, no long locks | Read-then-write check in Python (race condition) |
| D4 Money | NUMERIC(10,2) + Decimal; unit price copied to order\_item | No float rounding; history survives price changes | float; price looked up at read time |
| D5 Errors | One problem-JSON shape with stable `code` | UI maps codes to messages; tests assert codes | Free-text error strings |
| D6 Config | Environment variables via pydantic-settings | No secrets in Git (constitution M2) | Hard-coded connection string |
| D7 Frontend | Typed API client generated or hand-written from contract | Compile-time safety for field names | Untyped fetch calls scattered in components |

Reference checkout service (simplified):

```python
class OrderService:
    def __init__(self, session: Session, carts: CartRepository, orders: OrderRepository):
        self.session, self.carts, self.orders = session, carts, orders

    def checkout(self, customer_id: str) -> Order:
        with self.session.begin():                       # one transaction
            items = self.carts.list_for(customer_id)
            if not items:
                raise DomainError("CART_EMPTY", status=422)
            for item in items:
                updated = self.session.execute(
                    update(Book)
                    .where(Book.id == item.book_id, Book.stock >= item.quantity)
                    .values(stock=Book.stock - item.quantity)
                ).rowcount
                if updated == 0:                          # someone bought it first
                    raise DomainError("OUT_OF_STOCK", status=409, book_id=item.book_id)
            order = self.orders.create_from(customer_id, items)  # copies unit_price
            self.carts.clear(customer_id)
        return order
```

If any line fails, the whole transaction rolls back: no partial order, no lost stock. The 409 carries the book id so the UI can tell the customer which item sold out.

## Quality, governance and CI solution

No code reaches main unless four independent gates pass: the spec gate, the deterministic local guard, the CI pipeline and a human PR review.

| Test level | Tool | What it proves | Example |
| --- | --- | --- | --- |
| Unit | pytest | Service rules without HTTP/DB | Cart total = Σ price × qty |
| API / integration | pytest + HTTPX + test PostgreSQL | Contract, status codes, persistence | POST /books price=-5 → 422 |
| Concurrency | pytest with two threads | Stock never negative | AC-9.2 last copy, two buyers |
| UI end-to-end | Playwright | Real user journey in browser | list → cart → checkout → order status |

Governance controls in the reference repository:

- Constitution MUST rules M1–M6 are checked in PR review; a violation is a Blocker.
- GEMINI.md + standards keep the AI inside the agreed layering; .aiexclude keeps .env, keys and build output away from the model.
- Pre-commit guard runs ruff, pytest and a secret pattern scan; it blocks the commit whatever the AI produced.
- GitHub Actions runs lint, format check, pytest against a PostgreSQL service and the frontend build/tests on every PR.
- Branch protection requires green CI and one human approval; review findings are labelled Blocker / Major / Minor.
- evidence/traceability.md maps every FR → AC → task → commit → test; the capstone is scored on it (25% of the rubric).

Traceability excerpt from the reference solution:

| Requirement | Acceptance criterion | Task | Test |
| --- | --- | --- | --- |
| FR-003 Search | AC-3.1 | T012 | test\_search\_books\_by\_title |
| FR-005 Create | AC-5.2 | T015 | test\_create\_book\_rejects\_negative\_price |
| FR-009 Checkout | AC-9.1 | T031 | test\_checkout\_creates\_order\_and\_decrements\_stock |
| FR-009 Checkout | AC-9.2 | T032 | test\_concurrent\_checkout\_last\_copy |
| FR-010 Order status | AC-10.1 | T036 | e2e\_purchase\_journey.spec.ts |

## Delivery mapping to the 5-day TOC

The case study grows one increment per day and is complete, integrated and assessed by the end of Day 5.

| Day | Case-study increment | Requirements covered | Artifacts produced | Gate |
| --- | --- | --- | --- | --- |
| Day 1 | Baseline store + failure analysis of the vibe-coded prototype | — | environment-checklist, failure-log, GEMINI.md, standards, .aiexclude | Starters run; context loads |
| Day 2 | Rules and Catalog specification | FR-001–006, NFR-01–06 | constitution.md, specs/catalog/spec.md | Constitution + spec approved |
| Day 3 | Catalog designed and built | FR-001–006 | plan, contracts, data model, tasks, API, UI, tests | Catalog E2E smoke green |
| Day 4 | Cart/Orders built by agents, containerized | FR-007–010 | specs/orders/, agents, command, guard, docker-compose.yml | Concurrency test + compose up |
| Day 5 | Integrated store through CI and PR | All | ci.yml, PR review, test reports, traceability | Capstone demo scored |

Trainer note: keep a reference branch per day (day1-end … day5-end) so any team that falls behind can resume from the previous gate without losing the rest of the case study.
