# gin-web-rule Design

## Goal

Create a reusable skill at `skills/gin-web-rule` that guides Codex to produce and review company-style Gin Web projects with consistent architectural boundaries, API conventions, middleware behavior, configuration, security, database usage, and testing practices.

The skill should not act as a translation of Gin official documentation. It should act as an engineering rulebook for team collaboration and code generation.

## Scope

The skill targets Gin Web service development with the following default stack:

- `gin`
- `gorm`
- `go-playground/validator`
- `zap`
- `viper`
- `jwt`

The skill should apply when creating or modifying:

- Gin project scaffolds
- route registration and handler code
- service and repository layers
- request/response contracts
- middleware chains
- configuration bootstrap logic
- database access code
- authentication and authorization code
- unit tests and code review checklists

## Non-Goals

- Do not cover every optional Gin ecosystem package.
- Do not provide a runnable project template or code generator in this iteration.
- Do not replace the general-purpose `go-rule` skill; instead, complement it with Gin Web project constraints.

## Skill Shape

Create a new skill folder:

- `skills/gin-web-rule/SKILL.md`
- `skills/gin-web-rule/references/*.md`

`SKILL.md` stays concise and focuses on:

- trigger conditions
- quick reference
- core principles
- topic-to-reference routing

Detailed guidance lives in `references/` so Codex can load only the relevant document for the current task.

## Reference Set

Create these reference documents:

1. `project-structure.md`
2. `routing-and-handler.md`
3. `request-and-response.md`
4. `validation.md`
5. `middleware.md`
6. `error-handling.md`
7. `database-with-gorm.md`
8. `config-and-bootstrap.md`
9. `auth-and-security.md`
10. `logging-testing-and-review.md`

## Architectural Rules

The skill should enforce these default boundaries:

- `handler` only parses requests, invokes services, and writes responses.
- `service` owns business use-case orchestration.
- `repository` owns database access only.
- `model` represents persistence-layer entities.
- `middleware` owns cross-cutting concerns such as recovery, request tracing, authentication, and logging.
- `config/bootstrap` owns configuration loading and dependency initialization.

The skill should also require:

- separation between DTOs, persistence models, and response models
- a unified response envelope and error format
- centralized dependency injection during bootstrap
- end-to-end `context.Context` propagation
- standardized route naming and API style

## Strong Constraints

The skill should use strong normative wording such as `must`, `should`, and `must not`. It should read as a team engineering standard rather than optional advice.

Explicit prohibitions should include:

- business logic in handlers
- business rules in repositories
- returning GORM entities directly to clients
- hardcoded configuration or JWT secrets
- ad hoc error JSON formats
- unstructured logging only
- skipping validation failure handling
- long-running external calls inside DB transactions
- unconstrained large-table queries

## Reference Content Expectations

Each reference file should include:

- the intent of that topic
- mandatory rules
- recommended practices
- explicit anti-patterns
- short code examples only where they reduce ambiguity

The content should stay practical and concise, matching the style used by existing local skills such as `go-rule` and `go-zero-rule`.

## Validation Plan

After creating the skill:

1. run the local skill validator against `skills/gin-web-rule`
2. review the generated files for frontmatter accuracy and triggering clarity
3. verify the reference split is coherent and does not duplicate large blocks unnecessarily

## Open Decisions Resolved

These decisions were confirmed during brainstorming:

- The skill is for `Gin Web 项目规范`, not only raw Gin APIs.
- The target shape is a `公司级完整脚手架规范`.
- The default stack is `Gin + GORM + validator + Zap + Viper + JWT`.
- The writing style should be strong and prescriptive.

## Implementation Notes

- Prefer Chinese content because the surrounding skills in this repository are written primarily in Chinese.
- Keep the skill compatible with the existing repository style and naming conventions.
- Place the completed skill under `skills/` in this repository rather than in a user-global Codex skills directory, because the user explicitly requested repository-local placement.
