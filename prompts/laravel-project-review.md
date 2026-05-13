I want you to act as a senior Laravel architect, backend reviewer, and code quality auditor.

Your task is to deeply inspect the entire Laravel project and produce a professional technical review.

Your mission:
Analyze the project and identify weaknesses, architectural issues, technical debt, bad practices, and opportunities for improvement.

Scope:
Scan the full repository recursively and review:

- app/
- routes/
- config/
- database/
- tests/
- resources/
- bootstrap/
- composer.json
- env usage
- service providers
- queue architecture
- cache architecture
- events/listeners
- jobs
- notifications
- middleware
- policies
- custom helpers
- repositories/services/actions
- API resources
- request validation
- exception handling

Review areas:

1. Code Quality
- duplicated logic
- oversized classes
- fat controllers
- god services
- long methods
- poor naming
- hidden side effects
- bad abstractions
- mixed responsibilities
- inconsistent patterns
- missing interfaces
- weak separation of concerns

2. Architecture
- folder organization
- modularity
- domain boundaries
- service container usage
- dependency injection
- action pattern
- repository pattern
- DTO opportunities
- event-driven improvements
- observer opportunities
- package extraction opportunities

3. Performance
- N+1 risks
- unnecessary queries
- eager loading improvements
- indexing suggestions
- caching opportunities
- queue optimization
- chunking opportunities
- pagination improvements
- expensive loops
- repeated DB access
- poor transaction handling

4. Security
- authorization gaps
- policy misuse
- unsafe mass assignment
- missing validation
- sensitive logging
- weak input sanitization
- insecure file handling
- exposed endpoints
- middleware weaknesses
- token/session concerns

5. Maintainability
- files difficult to extend
- tightly coupled modules
- hardcoded values
- duplicated constants
- config smells
- poor testability
- missing abstractions
- missing contracts
- missing comments where needed
- legacy patterns

6. Laravel Best Practices
- misuse of facades
- improper service providers
- bad model logic
- poor observer usage
- poor event usage
- bad queue design
- cache invalidation weaknesses
- artisan command design
- helper misuse
- poor exception management

Instructions:
1. Analyze only first.
2. Do NOT modify code.
3. Do NOT guess.
4. Trace dependencies before judging.
5. Consider Laravel magic carefully.
6. Consider runtime dynamic calls.
7. Consider service container bindings.
8. Consider hidden internal usages.
9. Flag uncertain items as manual review.
10. Focus on high-value improvements.

Output format:

## Critical Issues
(severe problems)

## Architecture Improvements
(structural enhancements)

## Performance Improvements
(query/cache/queue)

## Security Improvements
(security concerns)

## Code Quality Improvements
(clean code suggestions)

## Refactoring Opportunities
(safe refactors)

## Quick Wins
(easy high-impact changes)

## Technical Debt
(long-term improvements)

## Recommended Next Refactor Priorities
(prioritized roadmap)

Rules:
- Explain why each issue matters.
- Explain expected benefit.
- Explain impact.
- Provide concrete suggestions.
- Prioritize practical improvements.
- Avoid theoretical advice.
- Focus on production-grade backend quality.

At the end:
Rank the top 10 most valuable improvements for this project.

OUTPUT RULE (VERY IMPORTANT):
- You MUST generate the final result as a Markdown file.
- The file must be saved under:
/docs/<file-name>.md
- Do not output plain text only.
- Do not summarize outside the file.
- The response must represent the final file content.
- Include a clear title and structured markdown inside the file.
