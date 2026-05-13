I want you to act as a senior Laravel codebase auditor and refactoring engineer.

Your task is to deeply inspect the entire Laravel project and identify all dead code, unused files, and unnecessary code safely.

Scope:
- Scan the full codebase recursively.
- Detect any:
  - Unused PHP files
  - Unused classes
  - Unused methods/functions
  - Unused services
  - Unused repositories
  - Unused traits
  - Unused helper functions
  - Unused jobs
  - Unused events/listeners
  - Unused requests/resources
  - Unused controllers/routes
  - Unused migrations references
  - Unused config entries
  - Unused imports (use statements)
  - Unused composer dependencies
  - Unused environment variables
  - Unreachable code blocks
  - Legacy commented code that should be removed

Instructions:
1. Analyze first, do NOT modify anything immediately.
2. Build dependency tracing:
   - route → controller
   - controller → service/action
   - service → model
   - event → listener
   - job → dispatch source
   - helper → call sites
3. Search for dynamic usage:
   - app()
   - resolve()
   - container bindings
   - reflection
   - facades
   - queued jobs
   - event dispatching
   - model observers
   - policies
   - gates
   - middleware
   - service providers
4. Be very careful with Laravel magic:
   - boot methods
   - accessors/mutators
   - scopes
   - casts
   - observers
   - artisan commands
   - macros
   - traits used indirectly
   - model relationships
5. Ignore vendor directory.
6. Ignore storage generated files.
7. Ignore node_modules.

Output format:
Create a markdown report with sections:

## Unused Files
(path + why)

## Unused Classes
(class + why)

## Unused Methods
(method + references)

## Suspicious But Needs Manual Review
(items that may be dynamically used)

## Safe To Delete Immediately
(only confirmed dead code)

IMPORTANT:
- Never guess.
- If uncertain, mark as "manual review".
- Only mark code as removable if proven unreachable.
- Explain dependency chain for each decision.
- Prioritize zero-risk cleanup.

After the report:
Ask for confirmation before deleting anything.

Do not delete code automatically.
Only prepare the audit report first.

OUTPUT RULE (VERY IMPORTANT):
- You MUST generate the final result as a Markdown file.
- The file must be saved under:
/docs/<file-name>.md
- Do not output plain text only.
- Do not summarize outside the file.
- The response must represent the final file content.
- Include a clear title and structured markdown inside the file.
