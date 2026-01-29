# a2z-vide-code

# Rules Files for LLM-Powered IDEs

This repository uses **rules files** to provide **critical project context** to AI-powered editors and IDEs.  
All rules are centralized under the **`skills/` directory** to ensure consistency, discoverability, and easier maintenance.

These rules act as **persistent instructions** for Large Language Models (LLMs), helping them generate code that follows our **architecture, coding standards, and best practices**.

---

Modern AI-assisted editors (LLM-powered IDEs) support **rules or instruction files** that provide **persistent, project-specific context** to the model.  
These files act as **guardrails**, helping the LLM understand your **coding standards, architecture, constraints, and best practices**—resulting in more accurate, consistent, and production-safe suggestions.

---

## Why Rules Files Matter

Rules files allow you to:

- Enforce **team-wide coding standards**
- Communicate **architecture decisions** and constraints
- Reduce repetitive prompt writing
- Prevent unsafe or non-compliant code generation
- Improve consistency across contributors and tools

Think of them as a **“system prompt for your repository.”**

---

## Supported IDEs and Their Rules Files

Different IDEs and AI tools expect rules in **specific filenames and locations**.  
Below is a consolidated reference.

| Environment / IDE        | Rules File Name              | Purpose |
|--------------------------|------------------------------|--------|
| **Firebase Studio**      | `airules.md`                 | Provide architectural and coding rules for Firebase Studio’s AI features |
| **Copilot-powered IDEs** | `copilot-instructions.md`    | Guide GitHub Copilot with repository-specific instructions |
| **Cursor**               | `cursorrules.md`             | Define Cursor-specific AI behavior and coding constraints |
| **JetBrains IDEs**       | `guidelines.md`              | Provide rules for AI assistants inside JetBrains IDEs |
| **VS Code**              | `.instructions.md`           | Configure AI behavior and coding standards in VS Code |
| **Windsurf**             | `guidelines.md`              | Define AI rules and best practices for Windsurf |

---

## Installation & Configuration

Each environment requires the rules file to be placed in a **specific location** within your project.

### Firebase Studio
- **File:** `airules.md`
- **Purpose:** Supplies architectural and coding context to Firebase Studio AI
- **Setup:** Follow Firebase Studio’s configuration steps for rules files

---

### Copilot-Powered IDEs
- **File:** `.github/copilot-instructions.md`
- **Purpose:** Customizes GitHub Copilot behavior at the repository level
- **Setup:**
  - Create `.github/` directory if it doesn’t exist
  - Add `copilot-instructions.md`
  - Commit to the repository

---

### Cursor
- **File:** `cursorrules.md`
- **Purpose:** Defines how Cursor’s AI should generate and refactor code
- **Setup:**
  - Place `cursorrules.md` in the project root
  - Cursor automatically detects it

---

### JetBrains IDEs
- **File:** `guidelines.md`
- **Purpose:** Provides coding and architectural rules for AI features
- **Setup:**
  - Place `guidelines.md` in the recommended project location
  - Ensure AI features are enabled in the IDE

---

### VS Code
- **File:** `.instructions.md`
- **Purpose:** Guides AI tools in VS Code with project-specific rules
- **Setup:**
  - Add `.instructions.md` to the project root
  - VS Code AI extensions will pick it up automatically

---

### Windsurf
- **File:** `guidelines.md`
- **Purpose:** Defines rules and best practices for Windsurf’s AI engine
- **Setup:**
  - Add `guidelines.md` to the project
  - Follow Windsurf’s configuration documentation

---

## What to Put in a Rules File

A well-written rules file typically includes:

- Language and framework best practices
- Architectural boundaries and layering rules
- Security and performance constraints
- Naming conventions and folder structure
- Patterns to prefer or avoid
- Accessibility and compliance requirements
- Explicit **DO / DO NOT** lists

---

## Best Practices for Rules Files

- Keep rules **clear and explicit**
- Prefer **bullet points and sections**
- Avoid vague language
- Update rules as the architecture evolves
- Treat rules files as **versioned documentation**

---

## Final Thought

> **Rules files turn AI from a generic code generator into a project-aware teammate.**

If you maintain multiple repositories or teams, standardizing these files can dramatically improve **code quality, consistency, and velocity**.


