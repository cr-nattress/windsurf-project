Windsurf Rules File Folder Structure
Overview
Windsurf uses a hierarchical structure for managing rules that guide the Cascade AI assistant. There are two main levels: Global Rules and Workspace Rules, each with specific file locations and organizational patterns.

Global Rules Structure
Location
Global rules are stored in a single file at the system level:

text
~/.codeium/windsurf/memories/global_rules.md
Windows: C:\Users$$USERNAME]\.codeium\windsurf\memories\global_rules.md
macOS/Linux: ~/.codeium/windsurf/memories/global_rules.md

Characteristics
Single file: All global rules are contained in one markdown file

Cross-workspace: Applied across all workspaces and projects

Character limit: 12,000 characters maximum

Format: Markdown (.md) file

Workspace Rules Structure
Location
Workspace rules are organized in a dedicated directory structure within each project:

text
your-project/
├── .windsurf/
│   └── rules/
│       ├── rule1.md
│       ├── rule2.md
│       └── custom-rule.md
Modern Structure (Wave 8+)
Since Wave 8, Windsurf has adopted a file-based rules system with the following structure:

text
project-root/
├── .windsurf/
│   └── rules/
│       ├── component-creation.md
│       ├── python-style-guide.md
│       ├── testing-standards.md
│       └── api-guidelines.md
├── .git/
└── [other project files]
Legacy Structure (Pre-Wave 8)
Before Wave 8, workspace rules used a single file approach:

text
project-root/
├── .windsurfrules
└── [other project files]
File Organization Best Practices
Naming Conventions
Use descriptive filenames that clearly indicate the rule's purpose

Follow kebab-case naming: python-style-guide.md

Group related rules with prefixes: frontend-styling.md, backend-api.md

File Structure Example
text
.windsurf/
└── rules/
    ├── coding-standards.md
    ├── project-specific.md
    ├── framework-rules.md
    ├── testing-guidelines.md
    └── deployment-rules.md
Content Organization
Each rule file should be structured as markdown with clear sections:

text
# Rule Title

## Description
Brief description of what this rule covers

## Guidelines
- Use bullet points for clarity
- Keep rules concise and specific
- Group related rules together

<coding_guidelines>
- My project's programming language is Python
- Use early returns when possible
- Always add documentation for new functions
</coding_guidelines>
Activation Modes
Each rule file can be configured with different activation modes:

Manual: Activated via @mention in Cascade

Always On: Applied to every Cascade action

Model Decision: AI decides relevance based on description

Glob: Applied based on file pattern matching (e.g., *.py, src/**/*.ts)

Key Differences Between Versions
Global Rules
Location: System-wide in ~/.codeium/windsurf/memories/

Scope: All workspaces

File: Single global_rules.md file

Access: Via Windsurf Settings → Global Rules

Workspace Rules
Location: Project-specific in .windsurf/rules/

Scope: Individual workspace/project

Files: Multiple markdown files per rule

Access: Via Windsurf Settings → Workspace Rules

Important Notes
Git Integration
The .windsurf/rules/ directory should be tracked in version control to share rules across team members. However, ensure that the workspace rules directory is positioned correctly relative to the Git repository root.

Character Limits
Individual rule files: 6,000 characters maximum

Combined global and workspace rules: 12,000 characters total

Rules exceeding limits are truncated

File Dependencies
Rules must be placed in the correct location relative to the Git repository root. If the .git folder is in a parent directory, ensure the .windsurf/rules/ folder is at the same level as the Git repository.

This structure provides a flexible, organized approach to managing AI rules that can be customized for different projects while maintaining consistency across your development workflow.