## Using github mcp and jira mcp

## 🎯 Task Implementation from JIRA

### 📌 Inputs
- JIRA Issue: {{ISSUE_KEY}}
- Git Branch: {{BRANCH_NAME}}

### 📌 Objective
Implement the feature described in JIRA issue **{{ISSUE_KEY}}**.

### 🛠️ Required Workflow

1. Use **jira MCP** to fetch the full issue details for **{{ISSUE_KEY}}**.
   - Read the description
   - Understand acceptance criteria
   - Identify edge cases
   - Extract technical requirements

2. Create a new git branch:

{{BRANCH_NAME}}

from the `dev` branch using **github MCP**.

3. Implement the required functionality according to the JIRA ticket.

### 🧑‍💻 Engineering Standards

- Follow project coding conventions
- Write clean and maintainable code
- Avoid unnecessary complexity
- Respect existing architecture
- Avoid breaking changes unless required

### 🧪 Testing

- Add or update **unit/integration tests** if needed
- Ensure all tests pass
- Validate feature behavior against JIRA acceptance criteria

### 🔍 Validation Checklist

- Feature works as expected
- No regressions introduced
- All tests passing
- Code formatted and linted

### 🚀 Deliverables

1. Commit changes using meaningful commit messages.
2. Push branch **{{BRANCH_NAME}}**.
3. Create a Pull Request targeting **dev**.

### 📝 Pull Request Requirements

PR must include:

- Clear description of implementation
- Reference JIRA issue **{{ISSUE_KEY}}**
- Any screenshots / logs if relevant
- Testing notes
- Mention edge cases handled

### ✅ Definition of Done

- Feature implemented correctly
- Tests added if needed
- Code follows project standards
- PR created and ready for review
