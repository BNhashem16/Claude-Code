---
name: cc-pr-release-notes
description: Generate structured release notes from a GitHub PR using MCP
---

## 📦 Generate Release Notes from GitHub PR

### 🎯 Objective
Use GitHub MCP tools to fetch and analyze the following Pull Request:
{{args}}

---

### 🛠️ Instructions
- Extract all relevant changes from the PR including commits, descriptions, and discussions.
- Understand the purpose and impact of the changes.
- Categorize the updates clearly.

---

### 📝 Output Format (Markdown)

Generate a clean and well-structured Markdown release note including the following sections:

#### 🚀 Features
List all new features introduced.

#### 🐛 Fixes
List bug fixes and resolved issues.

#### ⚡ Improvements
List enhancements, optimizations, or refactoring.

#### ⚠️ Breaking Changes
Clearly describe any breaking changes (if any).

#### 🔄 Migration Notes
Explain required actions for developers (DB changes, config updates, etc.).

#### 👤 End User Summary
Provide a simple, non-technical summary of what changed for end users.

---

### ✨ Guidelines
- Keep the tone professional, clear, and concise.
- Avoid unnecessary technical noise in the end-user summary.
- Use bullet points for readability.
- Ensure clean formatting and consistent section structure.
- Do not include irrelevant implementation details.

---

### 📦 Deliverable
- Generate the result as clean Markdown content
- Write the result to a file named `release-notes.md` in the current directory
- Overwrite the file if it exists
- Do NOT output the markdown in chat
- Only respond with: `release-notes.md created`