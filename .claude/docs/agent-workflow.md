Agent Workflow Methodology
Purpose
This document is a universal pre-task protocol for coding agents. Read it before every implementation task. The code is the last step, not the main event. Your job is to resolve units of understanding in dependency order — each layer small enough to get right — then compose from those resolved units.

The Unit Model
A "unit" is any atomic piece of understanding you must settle before reasoning correctly about the next layer. Units form a dependency chain. Building on an unresolved unit produces confident but wrong code.
Layer 0a — Structural Contracts The external truths you cannot invent. Database schemas, API specs (OpenAPI, GraphQL SDL), auth models, event/webhook payloads, config shapes. These are axioms. Read the files; do not infer the shapes.
Layer 0b — Exogenous Constraints Rules that originate outside the codebase — runtime limits, vendor requirements, tenancy isolation, idempotency mandates. These cannot be discovered from application code. They must be declared. If a constraints.md exists, load it. If not, extract them using the Signal Hierarchy below.
Layer 0c — Emergent Constraints State transitions, module boundaries, ownership rules. These can be partially reconstructed from code. Declared versions (if they exist) must be diffed against extraction. Discrepancies are either bugs or stale documentation — surface them before proceeding.
Layer 1 — Data Topology Given the contracts and constraints, map: what data is available where, who owns writes vs. reads, what the trust boundary is between layers. Place logic at the layer where its data already lives. This layer cannot be resolved until Layer 0 is settled.
Layer 2 — Reuse Map Given the topology, identify shared operations before writing a second caller that does the same work. Duplicated fetches, transforms, and validations are defects. Shared logic gets a deliberate home.
Layer 3 — Interface Design Given everything above, define the contract of each unit of work: inputs, outputs, error cases. The file, class, or function is an output of this contract, not the starting point.
Then write code.

The Four Documents
Every project should maintain a universal constraints.md at the project root. Larger codebases also have regional constraint files near the code they govern. The agent loads both.
Universal Documents (project root)
Document
Question It Answers
Contents
contracts.md
What are the structural axioms I cannot invent?
Database schema (entity shapes, fields, relations), API contracts (request/response signatures), auth model (identity, org structure, token shape), event payloads
constraints.md
What must the system not violate?
Runtime limits, tenancy isolation rules, idempotency requirements, state machines (or pointers to typed transition files), module boundaries (what each module does NOT own), ordering requirements (including cache-event ordering chains), enforced design rules (from pre-commit hooks, linters), intentionally public endpoints (allowlist for absence detection)
topology.md
Where does data live and how does it move?
Data flow from entry point to persistence, service call graph, integration boundaries, reuse inventory (what already exists that you must call, not rewrite)
design.md
What does it look and behave like?
Design tokens, component inventory, interaction conventions, branding rules. Only for products with a UI.

Regional Constraints (near the code)
Larger codebases have constraints that apply only to specific modules, not universally. These live in regional files (typically CLAUDE.md, README.md, or equivalent) adjacent to the code they govern.
Constraint hierarchy principle: Universal constraints are the floor, regional constraints are the ceiling. A regional file can add constraints ("this module also requires idempotent handlers") but never relaxes universal ones ("tenancy scoping is optional here"). If a regional file contradicts a universal constraint, that is a conflict to surface, not a permission to ignore.
Detecting the regional pattern. After Layer 3 (directory topology), the agent knows the codebase structure. The assessment sequence:
Check for existing regional files. find . -name "CLAUDE.md" -not -path "./CLAUDE.md" (or README.md, .constraints — any convention the project uses). If they exist, they are the regional constraint source. Read them for the module being worked on.
If no regional files exist, identify the module pattern. The codebase may still have module boundaries without regional documentation:
app/ with route groups → constraints may vary by route group (auth, data access patterns)
features/ or modules/ directories → constraints vary by domain feature
packages/ → monorepo, constraints vary by package (different runtimes, different dependencies)
services/ → service-oriented, constraints vary by service boundary
Flat structure → no regional constraints, everything is universal
If a module pattern exists but no regional files exist, flag the gap. Do not fill it automatically. Note: "this codebase has module boundaries at [pattern] but no regional constraint declarations. Universal constraints apply everywhere. Feature-specific constraints may be undocumented."
What regional constraints typically contain:
Which state machines this module owns (and pointers to the typed transition files)
Which collections/tables this module reads vs. writes
Which external services this module integrates with (and their specific constraints)
What this module is NOT responsible for (boundary declarations)
Module-specific ordering or idempotency requirements
Constraint loading sequence before every task:
Load constraints.md (universal) — always
Identify which module the current task touches (from Layer 3 topology or task description)
Search for regional constraint files in that module's directory
If found, load as additive constraints
If not found, proceed with universal constraints only — flag the absence if module boundaries exist
When transitions are declared in code. If state machines are expressed as typed data structures (e.g., Record<Status, Status[]> in TypeScript, dict[Status, list[Status]] in Python, or an XState machine definition), they are Layer 0a structural contracts, not emergent constraints. No extraction needed, no duplication in constraints.md. Instead, constraints.md contains a pointer: "AgentJobs transitions: see src/lib/state/agent-job-transitions.ts." The typed source file is the single source of truth. The pointer ensures agents know where to look.
If a document doesn't exist, that is a gap to flag — not permission to skip the layer.

Signal Hierarchy: Zero-Context Discovery
When entering a codebase with no existing documentation, resolve context in this order. Each layer's findings determine the next layer's targets. No exploratory grepping.
Layer 0 — Root Manifest Detection
Action: Single directory listing of the project root.
Language identification:
File Found
Inference
package.json
JavaScript / TypeScript
pyproject.toml / setup.py / requirements.txt
Python
go.mod
Go
Cargo.toml
Rust
Gemfile
Ruby
pom.xml / build.gradle
Java / Kotlin
Multiple of the above
Monorepo — recurse per package
None of the above
Unknown — prompt user, cannot proceed

Read the manifest. Extract two things: the framework (from dependencies) and the run commands (from scripts). These determine what every subsequent layer looks like.
Layer 1 — Infrastructure Detection
Action: Still at root. Still just ls. Look for deployment and runtime config.
File Found
Inference
vercel.json
Vercel — check functions for edge vs. serverless, maxDuration for timeout
wrangler.toml
Cloudflare Workers — edge runtime, CPU limits, no Node.js APIs
serverless.yml
Serverless Framework — read provider.timeout, provider.memorySize
docker-compose.yml
Containerized — read services for service boundaries
fly.toml / render.yaml / railway.toml
Hosted container — check resource limits
.github/workflows/
CI — reveals test commands, deployment targets, environment gates
None of the above
Runtime constraints unknown — flag for human, proceed

This layer yields exogenous runtime constraints. They exist only here.
Layer 2 — Dependency-Based Service Detection
Action: Read the dependency list from the manifest file. Don't read application code yet — read dependency names.
Dependency
Constraint Class Unlocked
stripe / stripe-python
Idempotency — webhook handlers must be idempotent
boto3 / aws-sdk
S3 upload patterns, SQS ordering, Lambda cold starts
prisma / sqlalchemy / drizzle / django
ORM layer exists — grep targets now known
clerk / auth0 / firebase-admin
Auth is external — token validation is not internal
redis / celery / bullmq
Background task infrastructure — async execution context
openai / anthropic
Rate limits, token quotas, response latency constraints

Also read .env.example or .env.template if it exists. Every key is a service dependency the manifest didn't reveal.
Layer 3 — Directory Topology
Action: Single ls of major directories, one level deep. Not recursive.
Identify:
src/ vs. flat → deliberate organization or not
backend/ + frontend/ → split architecture, two constraint domains
packages/ → monorepo with independent services
migrations/ or prisma/ or drizzle/ → relational DB confirmed, schema is greppable
functions/ → serverless entry points
infra/ or terraform/ → infrastructure-as-code with constraint signals
Layer 4 — Schema and State Discovery
Now targeted grepping begins. Because you know the language and ORM, you know exactly what to grep for.
Goal: Find every field that represents state.
Stack
Grep Target
Python / SQLAlchemy
grep -r "Enum(" --include="*.py" -l
Python / Django
grep -r "choices=" --include="*.py" -l
TypeScript / Prisma
Read prisma/schema.prisma, look for enum blocks
TypeScript / Drizzle
grep -r "pgEnum|mysqlEnum" --include="*.ts" -l
TypeScript / Mongoose
grep -r "enum:" --include="*.ts" -l
Go
grep -r "iota" --include="*.go" -l

Each grep returns file paths. Read only those files.
Layer 5 — Transition Extraction
For each status field found in Layer 4, one targeted grep:
grep -rn "[status_field]" [path-to-feature] --include="*.[ext]" | grep -v "==" | grep -v "!="
Filter for write operations (assignments), not read comparisons. The conditional context 3-4 lines above each write is where the transition rule lives.
Output: A candidate transition map per entity. Present for human confirmation before treating as canonical.
Tool Resolution Step
After Layer 0 identifies the language and Layer 1 identifies the platform, resolve the extraction toolkit before proceeding.
AST and state extraction by language:
Language
AST Tool
State Transition Signal
Contract Signal
JavaScript
acorn / babel-parser / eslint rules
Variable name heuristics (status, state) on assignment — no type guidance
JSDoc @typedef, OpenAPI YAML if exists — often implicit
Python
ast module, astroid
Enum subclass definitions → trace all assignments
Pydantic BaseModel subclasses — well-defined, greppable
TypeScript
ts-morph
Assignments to fields of union or enum type — type-aware, not heuristic
Interfaces and types are the contracts — first-class
Swift
swift-syntax, SwiftLint
enum with associated values — canonical Swift state pattern
Codable protocol conformances
Kotlin
detekt
Sealed classes — canonical Kotlin state machine pattern
Data classes with serialization annotations

Platform-specific constraint classes:
Platform
Additional Constraint Classes
Full-stack web (Next.js, Nuxt, SvelteKit)
Client/server boundary (server code cannot be imported client-side). Three execution contexts (client, serverless, edge) with different constraint profiles coexisting in one codebase.
Mobile (any)
Platform lifecycle (foreground/background/suspended — each permits different operations). Background execution hard limits (iOS: ~30s). Permission declarations as ordering constraints (must request before use). App store guidelines as exogenous constraints (binary size, privacy labels, entitlements).
Web frontend (any)
Core Web Vitals and WCAG accessibility targets if the team treats them as requirements, not aspirations. Measurable via Lighthouse and axe-core.
Python backend
Async/sync boundary — sync calls in async contexts are constraint violations.
API service (any)
Rate limiting, pagination contracts, versioning strategy.

Validation tools by platform:
Platform
Structural Validation
UI Validation
E2E
Python backend
mypy / pyright
None
pytest with HTTP client
TypeScript backend
tsc --noEmit
None
supertest or equivalent
Web frontend
tsc --noEmit
responsive checks, axe-core
Playwright / Cypress
Mobile (React Native)
tsc --noEmit
Snapshot testing, device simulator
Detox / Maestro
Mobile (Swift)
Swift compiler
XCUITest, SnapshotTesting
XCUITest
Mobile (Kotlin)
Kotlin compiler + detekt
Paparazzi, Espresso
Espresso

Unknown or Unique Codebase Fallback
When the language, framework, or stack isn't in the lookup tables, the Signal Hierarchy still works — Layers 0-3 are language-agnostic. What breaks is only the specific grep patterns and tool selections at Layers 4-5.
The fallback protocol:
Derive the state pattern from the language's idioms. Every language has a canonical way to represent finite states. Read 2-3 model/entity files found in Layer 3 topology scanning. Look for: enums, string unions, constant groups, sealed types, or any field with a small fixed set of values. That pattern becomes your grep target.
Derive the contract pattern from the dependency. If the ORM/framework is unknown, read its import statements and model definitions in the files you already found. The shape of the model declaration tells you how contracts are expressed. The syntax varies; the role is the same.
Use generic AST-free extraction. When no AST tool exists for the language:
grep -rn "=" [model-dirs] --include="*.[ext]" filtered to status-like field names found in step 1
Read 5 lines of context above each write (grep -B5) for conditional logic
Confidence is lower. Flag explicitly: "extraction confidence: medium — no AST tool available, heuristic grep only."
Read tests as constraint documentation. In any language, test files that assert "should not allow X" or "must require Y before Z" are machine-readable constraint declarations. grep -rn "should\|must\|cannot\|forbidden\|invalid" [test-dirs] surfaces these regardless of language.
Read CI/CD as constraint enforcement. Linting rules, pre-commit hooks, CI check steps — these encode what the team decided to enforce automatically. Every CI config is YAML or similar, and every check name hints at a constraint.
The key rule: When lookup tables don't match, shift from "look up the answer" to "derive the answer from what Layers 0-2 already revealed." The methodology degrades gracefully — extraction becomes less precise but never impossible. Always flag reduced confidence explicitly.
Blind Spot Detection
After extraction (whether by grep or by tooling), check for four categories of hidden constraints that static analysis misses by default.
Middleware blind spot. Auth, tenancy scoping, and rate limiting often live in middleware that is transparent to handlers. The handler code shows no trace of these constraints. Detection strategy: check whether the project has a middleware directory or request pipeline config. If auth dependencies exist on some endpoints but not others (detectable via AST extraction of Depends() patterns or equivalent), flag uncovered endpoints as potential tenancy violations. Absence detection caveat: an endpoint without auth could be intentionally public (health check, webhook receiver, public API) or accidentally exposed. The graph cannot distinguish these. To prevent false positives: constraints.md should declare intentionally public endpoints under a public-endpoints section. The inference query compares findings against that allowlist and only flags what is genuinely uncovered. Without this calibration, the compliance map produces false positives that erode trust in the entire output.
Database-layer blind spot. Triggers, stored procedures, row-level security policies, check constraints, and unique indexes enforce business rules with no trace in application code. Detection strategy: scan migration files for CREATE TRIGGER, CREATE POLICY, ALTER TABLE ... ADD CONSTRAINT, RLS, or equivalent DDL. Each one is a constraint that belongs in constraints.md even though application code doesn't reference it. Defense-in-depth classification: the migration scan also reveals how many layers each constraint is enforced at. If RLS policies exist, the application-level tenancy constraint has a database backstop — a bug in application code won't produce a cross-tenant read because the database rejects it. If they don't exist, the application-level constraint is the only enforcement layer. The output should classify each constraint as "app-layer only" or "also enforced at DB layer." Single-layer enforcement is higher risk and should flag for extra validation attention in constraints.md.
Generated code blind spot. If the project uses code generation (ORM client generation, API type generation, GraphQL hook generation), the generated output is not the source of truth — the generator config is. Reading generated files produces an incomplete or misleading model. Detection strategy: scan for # generated, // DO NOT EDIT, @generated, or auto-generated headers in files. When found, trace back to the generator config file and treat that as the contract source. Flag generated files as "not authoritative."
Runtime-only constraints. Redis TTLs set programmatically, connection pool sizes, task queue timeouts configured in infrastructure — these cannot be surfaced by any static analysis. Detection strategy: when a constraint type is expected (e.g., cache operations exist but no TTL is declared, or a task queue is used but no timeout is specified), output an explicit flag: [VALUE UNKNOWN — human confirmation required] rather than omitting the entry. The human fills the gap; the agent marks where the gap is.
Cache-event ordering blind spot. Cache staleness is not just a freshness problem — it is an ordering constraint. When a write operation is followed by an event emission (webhook dispatch, SSE broadcast, pub/sub message), and the event's consumer reads a cached endpoint that serves the same data, the consumer reads pre-write data regardless of how correct the event timing is. The constraint chain is: write → cache TTL lag → event consumer reads stale data. Detection strategy: for every write operation that is followed by an event emission in the same code path, check whether any subscriber of that event reads from a cached endpoint serving the written collection. If so, the cache TTL is an ordering constraint on the event — the consumer cannot trust the read until TTL expires or the cache is explicitly invalidated. This three-hop chain (write → cache → event consumer) is the actual risk surface, not the TTL number alone.
Tool-accelerated blind spot detection. When a code graph or AST analyzer exists, blind spot detection becomes a query rather than a scan:
Auth/tenancy: query all endpoints, check which have auth dependency annotations vs. which don't, compare against the declared public-endpoints allowlist → tenancy compliance map
Cache-event ordering: query all write operations that share a code path with an event emission, cross-reference the event's subscribers against cached endpoints serving the same collection → ordering risk map
Vendor constraints: query all environment variables, match against known vendor patterns (payment provider → idempotency, object storage → upload pattern, AI provider → rate limits) → exogenous constraint candidates
Migration constraints: scan migration directory for DDL statements that create triggers, policies, or constraints → database-layer constraint inventory with defense-in-depth classification
Each finding carries a confidence level. Auth dependency absence is high confidence (clear signal, low false positive rate). Environment variable to vendor mapping is medium confidence (STRIPE_SECRET is certain; a generic API_KEY is ambiguous). Cache-event ordering chains are lower confidence (requires three-hop reasoning across the graph, edge cases exist). High-confidence findings fast-track into constraints.md. Low-confidence findings are flagged for explicit human verification.
Decision Branching
At each layer, branch:
Layer 0 → language identified? YES: continue. NO: prompt user.
Layer 1 → runtime detected? YES: declare runtime constraints. NO: flag unknown, proceed.
Layer 2 → ORM found? YES: continue to Layer 4. NO: skip state extraction.
Layer 3 → monorepo? YES: repeat Layers 0-2 per package. NO: single domain.
Layer 4 → status fields found? YES: continue to Layer 5. NO: state machines section empty, flag.
Layer 5 → write ops found? Build map. Conflicts → flag for human.
Never make a confident declaration about something you couldn't find evidence for. Missing signals produce explicit flags, not omissions.

Tool Acceleration
If the project provides specialized tools, they replace or short-circuit layers of the Signal Hierarchy.
Discovery accelerators (replace manual grepping):
Vector store / embeddings index → Query before grepping. If rich results, skip to synthesis. If thin, fall back to Signal Hierarchy.
Code graph (graph generator + graph API) → Replaces Layers 3-5 entirely. Use get_all_features() for topology, get_feature_slice() for per-feature contract and dependency discovery, get_trace_slice() for data flow tracing, get_view_preset("contract") for reads/writes filtering.
Hooks directory (.claude/hooks/, git hooks, CI checks) → Each hook is a machine-executable constraint declaration. Direct input to constraints.md.
Infrastructure analyzer → Extracts agent/command/skill/hook definitions and their cross-references. Feeds topology.md for the agent orchestration layer.
Extraction tools (surface emergent constraints):
AST analyzer (language-specific) → Extracts DB operations, cache patterns, event pub/sub, auth dependencies, environment variables, route definitions, and background tasks from source code. Auth dependencies (e.g., Depends() in FastAPI, auth() in Next.js middleware, @Authorized decorators) are tenancy constraint signals. Environment variables are exogenous constraint signals (vendor dependencies). Cache decorators with TTLs are staleness and ordering constraint signals.
Static checkers (async checkers, complexity analyzers, query pattern checkers) → Violations imply constraints. Each violation surfaces a rule the codebase enforces implicitly.
Validation tools (post-implementation, not pre-task):
Type checkers, UI checkers, responsive validators → These verify constraints were honored after code is written. They belong in the validation phase, not the discovery phase.
Graph-aware extraction path. When a code graph exists, the agent's discovery sequence changes. The specific API depends on the graph tool, but the pattern is consistent:
Query graph statistics → confirm graph is current, get layer counts
Query for all features or modules → replaces Layer 3 directory topology scanning
For the target feature: query for the feature's slice of the graph → replaces Layer 4 schema discovery, gives collections, external APIs, and call chains in one query
For any endpoint being modified: query for the trace from endpoint to persistence → replaces manual data flow tracing
Run blind spot detection queries on the graph → surfaces what the graph doesn't capture, which is exactly what constraints.md needs to declare manually
The graph replaces the mechanical extraction layers. The blind spots are what remain for human declaration or targeted tool enhancement.
Important distinction: A code graph built from import/export metadata (file-level dependencies) replaces Layers 3-4 (topology and schema discovery). A code graph built from semantic analysis (AST-level call chains, data flow, type information) also replaces Layer 5 (transition extraction). Know which kind you have before relying on it for transition extraction.

Three Scenarios
Identify which scenario applies, then follow that protocol.
Greenfield: Declaration-First
Constraints are declared at decision time, not extracted later. When an architectural decision is made — execution environment, vendor integration, data model — the constraint goes into constraints.md immediately. State machines are declared when entities are designed. Transition files are written alongside the schema. The agent reads documents before every task and updates them when a new decision is made. No extraction needed.
Brownfield (No Documentation): Extraction Sprint
One-time bootstrap. Decompose by feature boundary for discovery, by constraint type for synthesis.
Discovery wave (parallel): One agent per feature directory. Each agent produces a structured extraction report:
All enum/status fields + every write site + conditional context → state transition candidates
All cross-module calls → boundary signals
All queries checked for tenant scoping → tenancy compliance map
Simultaneously:
One agent scans infrastructure config → runtime constraints
One agent reads vendor integration code → idempotency and routing constraints
Synthesis wave (single agent): Reads all discovery reports. Deduplicates. Surfaces conflicts. Produces first draft of the four documents.
Human confirmation (mandatory): Extraction produces a candidate, not ground truth. Some found transitions are bugs, not conventions. Human reviews, corrects, approves. After approval, documents are canonical.
Documentation Exists: Read → Act → Update
Read: Load documents before any planning begins. Resolve every relevant constraint before writing a single line.
Act: Implement with constraint awareness. When an implementation decision conflicts with a declared constraint, surface the conflict explicitly — do not resolve it silently.
Update: After every plan completes, run targeted extraction on modified files. Diff extraction against existing declarations. If new transitions, boundaries, or constraints appear in code that aren't declared, that is either a bug or a documentation gap. Update documents before closing the plan. This step belongs in the finalizer, not the builder's judgment — it runs unconditionally.

Pre-Implementation Checklist
Before writing any code, confirm:
Contracts resolved — Have I read every schema, API spec, and auth model this task touches?
Universal constraints loaded — Do I know the runtime limits, tenancy rules, idempotency requirements, and valid state transitions?
Regional constraints loaded — Have I checked for a CLAUDE.md or equivalent in the module I am working in? If it exists, have I loaded it? If the module has boundaries but no regional file, have I flagged this?
Topology mapped — Do I know where data lives, who owns writes, and what the trust boundary is between layers?
Reuse checked — Have I identified existing shared operations I must call rather than rewrite?
Interfaces defined — Have I specified inputs, outputs, and error cases for each unit of work before writing the body?
Failure modes clarified — For each operation: what can fail, must it be idempotent, what does the caller see on failure, is partial completion acceptable?
Boundaries confirmed — Do I know what this module is NOT responsible for?
If any answer is "no," resolve that unit before proceeding. Unresolved units produce confident but wrong code.




