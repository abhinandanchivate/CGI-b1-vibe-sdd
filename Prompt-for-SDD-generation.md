ROLE
You are a senior solution architect and Spec-Driven Development (SDD) practitioner using GitHub
Spec Kit. Produce a complete, implementation-ready SDD package for the case study attached as
@docs/case-study.md. The package must be consistent, traceable and testable end to end.

SOURCE OF TRUTH
- Use ONLY the attached case study. Do not invent features, fields, endpoints, error codes or rules.
- If something required is missing or ambiguous, write [NEEDS CLARIFICATION: <question>] and
  continue, then list every such question in a final "Open questions" section.
- Out-of-scope items in the case study must stay out of scope.

CONVENTIONS (apply everywhere)
- Requirements FR-0NN; acceptance criteria AC-<FR>.<n> in Given/When/Then; NFRs NFR-0N;
  tasks T0NN in ONE global sequence across all features; design decisions D<n>; constitution
  rules M1–M6 (MUST) and S1–S8 (SHOULD).
- Every AC states: HTTP status, problem `code` (from the error-code catalogue) and the resulting
  state (e.g. "stock is 3 and the cart is empty").
- Money: Decimal / NUMERIC(10,2) / JSON string with 2 decimals, INR. Lists paginated 20/50.
- Errors: RFC 9457 problem JSON {type, title, status, detail, code, instance, request_id}.
- Commit format: <type>(<scope>): <summary> [T0NN][AC-x.y].
- Organise delivery as Phase 1 Foundation, Phase 2 Specification, Phase 3 Catalog build,
  Phase 4 Orders build, Phase 5 Release. Tags gate-1-foundation … gate-5-release.
- Never use the words "day" (as a schedule unit) or "trainer". Roles are: product owner,
  architect, reviewer, delivery lead, developers, AI agents.

DELIVERABLES — produce each as a separate file block headed with its repository path
1.  .specify/memory/constitution.md — v1.1.0, M1–M6, S1–S8, governance, amendment history.
2.  .specify/templates/overrides/{spec,plan,tasks,checklist}-template.md — project templates
    that keep Spec Kit's core headings and placeholders and add PageTurn conventions.
3.  specs/000-shared/error-codes.md — every code, status, when raised, related ACs.
4.  specs/001-book-catalog/ — spec.md (user stories with priorities, all ACs, edge cases,
    requirements table, key entities, error codes used, NFRs touched, out of scope, success
    criteria, clarifications), checklists/requirements.md, plan.md (technical context,
    Constitution Check table with PASS/FAIL and evidence), research.md (decisions with rationale
    and rejected alternatives), data-model.md (tables, types, named constraints, indexes),
    contracts/openapi.yaml (OpenAPI 3.1, every status, Problem schema, money as string),
    quickstart.md (manual verification per AC), tasks.md (test-first phases, [P] and [USn]
    markers, each test task names its test functions and ACs).
5.  specs/002-cart-orders/ — the same artifact set. Checkout must use a conditional
    UPDATE … WHERE stock >= qty RETURNING price inside one transaction, locks taken in
    ascending book_id order, deadlock/serialization mapped to 409 CHECKOUT_CONFLICT; include
    AC-9.1 and AC-9.2 verbatim from the case study and a 25-run concurrency test.
6.  specs/003-ci-quality-gates/ — spec.md, plan.md, tasks.md for CI, branch protection,
    traceability generation.
7.  GEMINI.md, docs/standards.md, .aiexclude, .env.example, docs/agents/{test-author,
    backend-implementer,frontend-implementer,reviewer}.md.
8.  evidence/traceability.md — FR → AC → Task → Test for every AC (commit column left as
    "generated"), plus NFR → evidence mapping.
9.  A Definition of Ready, Definition of Done, risk register and phase-gate checklist.

QUALITY RULES
- Specs describe WHAT and WHY only; all technology goes in plan.md and later artifacts.
- Every FR has ≥ 1 AC; every AC has ≥ 1 task and ≥ 1 named test; every task traces to an AC
  or an NFR. No orphan tasks, no untested ACs.
- Tests are ordered before the implementation tasks that make them pass.
- Constitution Check must reference concrete evidence (file, test or CI job), not "yes".
- Contracts, data model, schemas and ACs must agree on every field name, type and status code.

SELF-CHECK BEFORE ANSWERING (show the result as a table at the end)
- AC count per FR; ACs without tasks; ACs without tests; tasks without ACs/NFRs.
- Error codes used but missing from the catalogue.
- Fields in contracts not present in the spec.
- Any use of the forbidden words.
Fix every issue found before giving the final output.

OUTPUT FORMAT
Start with a one-paragraph summary and a table of contents of all files. Then output each file
as:  ### <path>  followed by a fenced code block with its full content. End with
"Open questions" and the self-check table.
