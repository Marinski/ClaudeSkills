# Claude Skills Creation Guide

## Overview

### What are Claude Skills?

Claude Skills are specialized instruction sets that extend Claude's capabilities for specific tasks or domains. They work through a three-tier context loading system that optimizes performance and token usage:

1. **Metadata (Always Loaded)**: ~100 words - YAML frontmatter containing name, description, and optional configuration
2. **SKILL.md Body (Loaded When Activated)**: <5,000 words (~250-650 lines) - Core instructions and workflows
3. **Bundled Resources (Loaded On-Demand)**: No limit - Scripts, references, examples, and assets

This tiered approach ensures Claude only loads what's needed when it's needed, keeping context efficient while maintaining comprehensive capabilities.

### When to Create a Skill

Create a skill when you have:
- Specialized workflows that require consistent execution
- Domain-specific knowledge that needs to be activated on-demand
- Complex multi-step processes that benefit from structured guidance
- Reusable patterns that apply across multiple projects

## Quick Start Checklist

- [ ] Create directory with hyphen-case name: `my-skill-name/`
- [ ] Add `SKILL.md` with YAML frontmatter (name + description required)
- [ ] Write clear activation conditions in description
- [ ] Keep core instructions under 5,000 words
- [ ] Use imperative form for instructions: "Create", "Analyze", "Generate"
- [ ] Move detailed references to `references/` directory
- [ ] Move executable code to `scripts/` directory
- [ ] Move examples to `examples/` directory if more than 1-2 inline
- [ ] Test skill activation with clear trigger phrases
- [ ] Review quality checklist before publishing

## File Structure Standards

### Minimal Structure (Required)

```
skill-name/
└── SKILL.md                    # Core instructions with YAML frontmatter
```

### Standard Structure (Recommended)

```
skill-name/
├── SKILL.md                    # Core instructions (required)
├── scripts/                    # Executable code (optional)
│   ├── helper.py
│   └── validator.sh
├── references/                 # Detailed documentation (optional)
│   ├── api-reference.md
│   └── best-practices.md
├── examples/                   # Sample use cases (optional)
│   ├── basic-example.md
│   └── advanced-example.md
└── assets/                     # Templates, icons, fonts (optional)
    ├── template.json
    └── icon.svg
```

### When to Use Each Directory

**scripts/**
- Deterministic operations (parsing, validation, formatting)
- Reusable utility functions
- External API integrations
- File manipulation tools

**references/**
- Extensive API documentation (>100 lines)
- Technical specifications
- Style guides and standards
- Detailed best practices

**examples/**
- More than 2 complete examples
- Complex use case demonstrations
- Before/after comparisons
- Template variations

**assets/**
- Templates (JSON, YAML, config files)
- Visual resources (icons, logos)
- Fonts or design assets
- Data files or fixtures

## YAML Frontmatter Deep Dive

### Required Fields

```yaml
---
name: skill-name-in-hyphen-case
description: Clear explanation of what the skill does and when Claude should use it
---
```

**name**: Must match directory name exactly. Use hyphen-case (lowercase with hyphens).

**description**: This is critical - it determines when Claude activates your skill. Be specific about:
- What the skill does
- When it should be used
- What problems it solves
- Key capabilities

### Optional Fields

```yaml
---
name: advanced-skill
description: Complete description here
version: 1.0.0
license: MIT
allowed-tools: [Read, Write, Edit, Bash]
metadata:
  author: Your Name
  category: development
  tags: [testing, automation]
---
```

### Good vs Bad Descriptions

**Bad - Too Vague:**
```yaml
description: Helps with testing
```
Why bad: Claude won't know when to activate this.

**Bad - Too Generic:**
```yaml
description: A skill for working with web applications
```
Why bad: Not specific enough about capabilities or use cases.

**Good - Clear and Specific:**
```yaml
description: Comprehensive web application testing skill that guides systematic testing across functionality, UI/UX, performance, security, and accessibility. Use when testing web applications or creating test plans.
```
Why good: Explains what it does, what areas it covers, and when to use it.

**Good - Action-Oriented:**
```yaml
description: Generate professional PowerPoint presentations from outlines or content. Handles slide layouts, formatting, images, charts, and animations. Use when creating .pptx files or converting content to presentation format.
```
Why good: Lists specific capabilities and clear activation trigger.

### Real Examples from Official Skills

```yaml
# Simple skill
---
name: template-skill
description: A minimal template for creating new Claude skills
---

# Medium complexity
---
name: brand-guidelines
description: Apply consistent brand voice, visual identity, and communication standards across all content. Use when creating marketing materials, documentation, or public communications.
---

# Complex skill
---
name: mcp-builder
description: Build Model Context Protocol (MCP) servers with proper architecture, resource/tool/prompt implementations, and testing. Use when creating MCP servers or integrating external data sources with Claude.
---
```

## SKILL.md Content Guidelines

### Standard Sections

Most skills benefit from this structure:

```markdown
---
name: your-skill
description: Your clear description
---

# Skill Name

## Purpose
Brief overview of what this skill accomplishes.

## When to Use This Skill
Clear activation conditions and use cases.

## Core Instructions

### Step 1: [First Phase]
Detailed instructions...

### Step 2: [Second Phase]
Detailed instructions...

### Step 3: [Final Phase]
Detailed instructions...

## Key Principles
- Important guideline 1
- Important guideline 2
- Important guideline 3

## Common Patterns
Brief examples of typical usage (1-2 examples max).

## Additional Resources
- See `references/detailed-guide.md` for complete API documentation
- See `examples/` for more use cases
- See `scripts/helper.py` for validation utilities
```

### What to Include in SKILL.md

**DO Include:**
- Core workflow steps
- Essential principles and guidelines
- 1-2 inline examples (brief)
- Decision trees for common choices
- Key quality standards
- References to external files

**DO NOT Include:**
- Complete API documentation (move to references/)
- Multiple detailed examples (move to examples/)
- Full code files (move to scripts/)
- Extensive style guides (move to references/)
- Long tables or specifications (move to references/)

### Writing Style

**Use Imperative Form:**
```markdown
✅ Good:
- Create a new test suite
- Analyze the user requirements
- Generate documentation

❌ Bad:
- You should create a test suite
- The requirements need to be analyzed
- Documentation can be generated
```

**Be Objective and Direct:**
```markdown
✅ Good:
Execute tests in this order: unit tests first, then integration tests, then end-to-end tests.

❌ Bad:
I think it's probably best if you maybe run unit tests before the other ones.
```

**Use Clear Structure:**
```markdown
✅ Good:
## Testing Workflow

1. Analyze codebase structure
2. Identify test coverage gaps
3. Generate test cases
4. Implement tests
5. Verify coverage

❌ Bad:
Just test the code by looking at it and then write some tests for the parts that need them.
```

## Length Guidelines

Based on analysis of official Anthropic skills:

### Simple Skills: 50-120 lines

**When to use:**
- Single-purpose focused tasks
- Minimal decision trees
- Straightforward workflows
- Template or scaffold generation

**Examples from official skills:**
- `template-skill`: ~13 lines (absolute minimum)
- `internal-comms`: ~45-50 lines (simple multi-use)

**Structure:**
```markdown
---
frontmatter
---

# Brief introduction (2-5 lines)

## Core Instructions (30-80 lines)
- Step-by-step workflow
- Key principles
- 1 brief example

## Resources (5-10 lines)
- References to external files
```

### Medium Skills: 120-350 lines

**When to use:**
- Multi-phase workflows
- Moderate complexity
- Multiple decision points
- Several related capabilities

**Examples from official skills:**
- `brand-guidelines`: ~95 lines
- `webapp-testing`: ~110 lines
- `artifacts-builder`: ~120 lines

**Structure:**
```markdown
---
frontmatter
---

# Introduction (10-20 lines)

## Purpose and Activation (10-15 lines)

## Core Workflow (80-200 lines)
### Phase 1
### Phase 2
### Phase 3

## Key Principles (20-40 lines)

## Patterns and Examples (20-50 lines)

## Resources (10-20 lines)
```

### Complex Skills: 350-650 lines

**When to use:**
- Highly specialized domains
- Extensive workflows
- Multiple sub-systems
- Technical implementations

**Examples from official skills:**
- `pdf`: ~340 lines (format manipulation)
- `mcp-builder`: ~550 lines (technical architecture)
- `pptx`: ~650 lines (complex formatting)
- `algorithmic-art`: ~650+ lines (creative domain)

**Structure:**
```markdown
---
frontmatter
---

# Comprehensive Introduction (20-40 lines)

## Philosophy and Approach (30-60 lines)

## Core Workflow (200-400 lines)
### Major Phase 1
  #### Sub-phase A
  #### Sub-phase B
### Major Phase 2
### Major Phase 3

## Technical Specifications (50-100 lines)

## Quality Standards (40-80 lines)

## Common Patterns (30-60 lines)

## Troubleshooting (20-40 lines)

## Resources (10-20 lines)
```

### Going Beyond 650 Lines

If your skill exceeds 650 lines, apply progressive disclosure:

```markdown
# In SKILL.md (stays under 650 lines):
## API Integration
Use the REST API endpoints for data retrieval. See `references/api-documentation.md` for complete endpoint details, authentication, and error handling.

# In references/api-documentation.md (no length limit):
[Complete API documentation with all endpoints, parameters, examples...]
```

## Progressive Disclosure in Practice

### When to Split Content

**Keep in SKILL.md:**
- Essential workflow steps
- Core decision logic
- Critical quality standards
- Brief orienting examples

**Move to references/:**
- API documentation >100 lines
- Complete technical specifications
- Detailed style guides
- Comprehensive best practices

**Move to scripts/:**
- Validation functions
- Parsing utilities
- Format converters
- API wrappers

**Move to examples/:**
- More than 2 examples
- Complete use case demonstrations
- Variations and templates

### Reference Patterns

**Inline Reference (Good):**
```markdown
## API Authentication

Use OAuth 2.0 for authentication. See `references/api-auth.md` for:
- Token acquisition flow
- Refresh token handling
- Error scenarios
```

**Inline Reference (Better):**
```markdown
## API Authentication

Basic OAuth 2.0 flow:
1. Request authorization code
2. Exchange for access token
3. Include in Authorization header: `Bearer {token}`

For complete details including refresh tokens, error handling, and security best practices, see `references/api-auth.md`.
```

**Script Reference:**
```markdown
## Data Validation

Validate user input before processing:
- Check required fields
- Verify data types
- Sanitize strings

Use `scripts/validator.py` to automate validation:
```bash
python scripts/validator.py --input data.json --schema schema.json
```

**Example Reference:**
```markdown
## Common Use Cases

Basic newsletter format:
[Brief inline example]

For additional templates including promotional emails, announcements, and event invitations, see `examples/email-templates/`.
```

### Real Example: pdf Skill

The PDF skill (~340 lines) uses progressive disclosure effectively:

**In SKILL.md:**
- Core PDF manipulation workflow
- Key principles for text extraction
- Layout analysis guidelines
- Brief example of form filling

**Could be moved to references/ (if it grew larger):**
- Complete PDF specification details
- Extended OCR configuration options
- Comprehensive security settings

**Could use scripts/ for:**
- PDF validation utilities
- Batch processing helpers
- Format conversion tools

## Writing Style Guide

### Tone and Voice

**Objective and Instructional:**
```markdown
✅ Good:
Execute the following steps in sequence:
1. Analyze the input requirements
2. Generate the test structure
3. Implement test cases

❌ Bad:
So basically what you want to do is look at what the user needs and then maybe write some tests for it.
```

**Professional but Clear:**
```markdown
✅ Good:
Structure tests using the Arrange-Act-Assert pattern for clarity and maintainability.

❌ Bad:
You absolutely MUST use AAA pattern or everything will be a disaster!!!
```

**Specific and Actionable:**
```markdown
✅ Good:
Create unit tests with at least 80% code coverage, focusing on:
- Edge cases (boundary values, null inputs)
- Error conditions (exceptions, invalid states)
- Core business logic (critical paths)

❌ Bad:
Make sure you test the important stuff really well.
```

### Formatting Standards

**Use Headers Hierarchically:**
```markdown
# Skill Name (H1 - only once)

## Major Section (H2)

### Subsection (H3)

#### Minor Point (H4)
```

**Use Lists for Steps:**
```markdown
✅ Good:
1. First concrete action
2. Second concrete action
3. Third concrete action

❌ Bad:
Do the first thing, then do the second thing, and after that the third thing.
```

**Use Bullet Points for Principles:**
```markdown
✅ Good:
Key principles:
- Maintainability over cleverness
- Explicit over implicit
- Tested over assumed

❌ Bad:
The key principles are maintainability, being explicit, and testing things.
```

**Use Code Blocks Appropriately:**
```markdown
✅ Good:
Example test structure:
\`\`\`python
def test_user_creation():
    # Arrange
    user_data = {"name": "Test", "email": "test@example.com"}

    # Act
    user = create_user(user_data)

    # Assert
    assert user.name == "Test"
\`\`\`

❌ Bad:
You can write a test like test_user_creation() that creates a user and checks it.
```

### Third-Person for Descriptions

**In YAML descriptions:**
```yaml
✅ Good:
description: Generates comprehensive test suites for Python applications

❌ Bad:
description: I will help you write tests for your Python code
```

**In skill body:**
```markdown
✅ Good:
This skill guides the creation of professional presentations.

❌ Bad:
I'm a skill that will help you make presentations.
```

## Helper Scripts Best Practices

### When to Create Scripts

Create helper scripts for:
- Deterministic operations (parsing, formatting, validation)
- External API calls
- File system operations
- Complex calculations
- Repetitive tasks

**Example from mcp-builder skill:**
```
mcp-builder/
├── SKILL.md
└── scripts/
    ├── create_mcp_server.py       # Scaffold generator
    ├── validate_schema.py          # JSON schema validator
    └── test_server.sh              # Testing automation
```

### Script Structure

**Good Script Header:**
```python
#!/usr/bin/env python3
"""
PDF Metadata Extractor

Extracts metadata from PDF files including author, title, creation date,
and custom properties.

Usage:
    python extract_metadata.py input.pdf [--format json|yaml]

Returns:
    Structured metadata in specified format
"""
import sys
import argparse
# ... rest of implementation
```

**Script Documentation in SKILL.md:**
```markdown
## Metadata Extraction

Extract PDF metadata using the helper script:

\`\`\`bash
python scripts/extract_metadata.py document.pdf --format json
\`\`\`

This returns structured data including:
- Document properties (title, author, subject)
- Creation and modification dates
- PDF version and compatibility
- Custom metadata fields

See `scripts/extract_metadata.py --help` for all options.
```

### Script vs Inline Instructions

**Use Script When:**
```markdown
❌ Don't inline this:
To validate JSON schema:
1. Parse the JSON file
2. Load the schema definition
3. Check each field against schema types
4. Validate required fields
5. Check format constraints
6. Return validation errors

✅ Use script instead:
Validate data using: `python scripts/validator.py data.json schema.json`
```

**Use Inline When:**
```markdown
✅ Keep this inline:
To structure your test file:
1. Import test framework
2. Create test class
3. Write test methods starting with 'test_'
4. Use descriptive assertion messages

[This is high-level workflow guidance, not deterministic steps]
```

## Examples Directory Guide

### What Goes in examples/

**Good Candidates:**
- Complete use case demonstrations (>30 lines)
- Template variations (>3 variations)
- Before/after comparisons
- Complex configuration examples

**Poor Candidates:**
- Single brief examples (<20 lines) - keep inline
- Code snippets - keep inline
- Simple demonstrations - keep inline

### Example Organization

```
examples/
├── basic/
│   ├── simple-test.md
│   └── minimal-config.md
├── advanced/
│   ├── integration-test.md
│   └── complex-mocking.md
└── templates/
    ├── rest-api-tests.md
    ├── database-tests.md
    └── ui-tests.md
```

### Referencing Examples

**In SKILL.md:**
```markdown
## Quick Start Example

Basic test structure:
\`\`\`python
def test_addition():
    assert add(2, 2) == 4
\`\`\`

For complete examples including:
- REST API testing (`examples/templates/rest-api-tests.md`)
- Database testing (`examples/templates/database-tests.md`)
- UI testing (`examples/templates/ui-tests.md`)

See the `examples/` directory.
```

### Example File Structure

**Good Example File:**
```markdown
# REST API Testing Template

## Overview
This template demonstrates testing REST API endpoints with authentication,
error handling, and response validation.

## Setup
\`\`\`python
import pytest
import requests

BASE_URL = "https://api.example.com"
\`\`\`

## Authentication Tests
\`\`\`python
def test_login_success():
    # ... complete implementation
\`\`\`

## CRUD Operation Tests
\`\`\`python
def test_create_resource():
    # ... complete implementation
\`\`\`

## Error Handling Tests
\`\`\`python
def test_404_handling():
    # ... complete implementation
\`\`\`

## Usage Notes
- Modify BASE_URL for your environment
- Add authentication tokens in conftest.py
- Adjust timeouts based on API performance
```

## Common Patterns

### Pattern 1: Modular Multi-Use (internal-comms)

**When to use:** Skill has multiple distinct capabilities that might be used independently.

**Structure:**
```markdown
# Skill Name

## Purpose
[Brief overview]

## Usage Mode 1: [Capability A]
Instructions for first capability...

## Usage Mode 2: [Capability B]
Instructions for second capability...

## Usage Mode 3: [Capability C]
Instructions for third capability...

## General Principles
Principles that apply across all modes...
```

**Example:**
```markdown
# Internal Communications

## Usage Mode 1: Slack Messages
Craft clear, actionable Slack messages...

## Usage Mode 2: Email Communications
Write professional emails...

## Usage Mode 3: Announcement Posts
Create engaging announcements...
```

### Pattern 2: Format-Based Modularization (document-skills)

**When to use:** Skill focuses on specific file formats or output types.

**Structure:**
```markdown
# Format Handler

## Format Overview
Technical details about the format...

## Creation Workflow
1. Input analysis
2. Structure generation
3. Content population
4. Format validation

## Format-Specific Operations
### Operation A
### Operation B
### Operation C

## Quality Standards
Format-specific quality criteria...
```

**Example:**
```markdown
# PDF Skill

## PDF Document Structure
[Technical overview]

## Document Generation Workflow
1. Analyze content and requirements
2. Design document structure
3. Create PDF with appropriate layout
4. Validate output

## PDF Operations
### Text Extraction
### Form Filling
### Metadata Management
```

### Pattern 3: Progressive Complexity (pdf, webapp-testing)

**When to use:** Skill supports both simple and advanced use cases.

**Structure:**
```markdown
# Skill Name

## Quick Start
Brief instructions for simple use case...

## Standard Workflow
1. Basic step 1
2. Basic step 2
3. Basic step 3

## Advanced Capabilities
### Advanced Feature 1
Detailed instructions...

### Advanced Feature 2
Detailed instructions...

## Expert Techniques
Complex scenarios and optimization...
```

**Example:**
```markdown
# Web Application Testing

## Quick Start
Run basic smoke tests to verify core functionality.

## Standard Testing Workflow
1. Functionality testing
2. UI/UX testing
3. Basic performance checks

## Advanced Testing
### Security Testing
Comprehensive security audit...

### Performance Optimization
Load testing and profiling...
```

### Pattern 4: Workflow-Centric (artifacts-builder, mcp-builder)

**When to use:** Skill guides through complex multi-phase projects.

**Structure:**
```markdown
# Project Builder

## Phase 1: Planning and Design
### Planning Steps
### Design Considerations
### Deliverables

## Phase 2: Implementation
### Implementation Steps
### Code Standards
### Deliverables

## Phase 3: Testing and Validation
### Testing Steps
### Validation Criteria
### Deliverables

## Phase 4: Deployment
### Deployment Steps
### Verification
### Deliverables
```

**Example:**
```markdown
# MCP Server Builder

## Phase 1: Architecture Design
### Server Structure
### Resource Planning
### Tool Definition

## Phase 2: Core Implementation
### Server Bootstrap
### Resource Implementation
### Tool Implementation

## Phase 3: Testing
### Unit Tests
### Integration Tests
### Manual Verification
```

### Pattern 5: Philosophy-First (algorithmic-art)

**When to use:** Skill requires understanding principles before execution.

**Structure:**
```markdown
# Creative/Specialized Skill

## Philosophy and Principles
Core concepts and theory...

## Fundamental Techniques
Basic building blocks...

## Composition and Structure
How to combine techniques...

## Creation Workflow
1. Conceptualize
2. Design
3. Implement
4. Refine

## Advanced Concepts
Expert-level theory and practice...
```

**Example:**
```markdown
# Algorithmic Art

## Philosophy
The intersection of mathematics, randomness, and aesthetics...

## Fundamental Techniques
### Geometric Patterns
### Randomness and Noise
### Color Theory

## Composition Principles
Balance, rhythm, and visual flow...

## Creation Workflow
[Step-by-step process]
```

## Testing and Validation

### Manual Testing Checklist

Test your skill before publishing:

- [ ] **Activation Test**: Does Claude activate the skill with expected trigger phrases?
- [ ] **Workflow Test**: Does each step in the workflow execute correctly?
- [ ] **Resource Loading**: Do references to external files work correctly?
- [ ] **Script Execution**: Do helper scripts run without errors?
- [ ] **Example Validation**: Do provided examples work as documented?
- [ ] **Edge Cases**: Does the skill handle unexpected inputs gracefully?

### Testing Script Integration

```bash
# Test skill activation
echo "Testing skill activation..."

# Test script executability
if [ -d "scripts" ]; then
    for script in scripts/*; do
        if [ -f "$script" ]; then
            echo "Testing $script..."
            # Add appropriate test command
        fi
    done
fi

# Validate YAML frontmatter
echo "Validating SKILL.md frontmatter..."
# Add YAML validation

# Check file references
echo "Checking file references..."
grep -r "references/" SKILL.md
grep -r "examples/" SKILL.md
grep -r "scripts/" SKILL.md
```

### Integration Testing

Test the skill in real scenarios:

1. **Fresh Context**: Start new Claude session and trigger skill
2. **Multiple Activations**: Use skill multiple times in same session
3. **Resource Loading**: Verify external files load correctly
4. **Script Execution**: Test all helper scripts with real data
5. **Error Handling**: Test with invalid inputs or missing resources

### User Testing

Before finalizing:

1. Have someone else try to use the skill
2. Ask: "Is it clear when to use this skill?"
3. Ask: "Are the instructions easy to follow?"
4. Ask: "What's confusing or unclear?"
5. Iterate based on feedback

## Common Mistakes

### Mistake 1: Vague Activation Description

**Bad:**
```yaml
---
name: helper-skill
description: Helps with various tasks
---
```

**Why it's bad:** Claude won't know when to activate this skill.

**Good:**
```yaml
---
name: test-generator
description: Generate comprehensive unit and integration tests for Python applications. Use when creating test suites, adding test coverage, or implementing test-driven development.
---
```

### Mistake 2: Inline Documentation Overload

**Bad - 400 lines in SKILL.md:**
```markdown
# API Integration Skill

## Complete API Documentation

### Endpoint: /users
**Method:** GET
**Parameters:**
- id (integer, required): User ID
- include (string, optional): Related resources
[... 300 more lines of endpoint documentation ...]

### Endpoint: /posts
[... another 100 lines ...]
```

**Good - Progressive Disclosure:**
```markdown
# API Integration Skill

## API Overview
This skill integrates with the REST API using standard HTTP methods.

### Common Endpoints
- `/users` - User management
- `/posts` - Content operations
- `/auth` - Authentication

For complete endpoint documentation, parameters, and response formats, see `references/api-documentation.md`.

## Integration Workflow
1. Authenticate using OAuth 2.0
2. Make API requests with proper headers
3. Handle responses and errors
4. Implement retry logic for transient failures

[Continue with workflow instructions...]
```

### Mistake 3: Missing Imperative Voice

**Bad:**
```markdown
The user should start by analyzing the requirements. Then they can create
the structure. After that, the implementation should be done.
```

**Good:**
```markdown
1. Analyze the requirements
2. Create the structure
3. Implement the solution
```

### Mistake 4: Excessive Formatting

**Bad - Over-formatted:**
```markdown
<div align="center">

# 🎨 AMAZING SKILL 🎨

**The Most Incredible Skill Ever Created**

---

✨ **FEATURING** ✨

</div>

<div align="center">
  <table>
    <tr>
      <td align="center">Feature 1</td>
      <td align="center">Feature 2</td>
    </tr>
  </table>
</div>
```

**Good - Clean and Functional:**
```markdown
# Amazing Skill

Professional skill for [specific purpose].

## Key Features
- Feature 1: [description]
- Feature 2: [description]
```

### Mistake 5: Script-Appropriate Tasks Inline

**Bad:**
```markdown
## JSON Validation Process

To validate JSON:
1. Read the file contents
2. Parse the JSON string
3. Check if 'name' field exists and is a string
4. Check if 'age' field exists and is an integer
5. Check if 'email' field exists and matches email regex pattern
6. Check if 'status' is one of: 'active', 'inactive', 'pending'
7. Return list of validation errors

[20 more lines of validation rules...]
```

**Good:**
```markdown
## JSON Validation

Validate data structure using the schema validator:

\`\`\`bash
python scripts/validate_json.py data.json --schema schema.json
\`\`\`

The validator checks:
- Required fields presence and types
- Format constraints (email, URL, date)
- Enum value restrictions
- Custom business rules

See `scripts/validate_json.py --help` for all options.
```

### Mistake 6: No Clear Structure

**Bad:**
```markdown
# Testing Skill

So basically this skill helps you write tests. You can test APIs and
also test UIs and databases too. Make sure tests are good quality.
Sometimes you need mocking and sometimes you don't. Error handling
is important. Also think about edge cases.
```

**Good:**
```markdown
# Testing Skill

## Purpose
Generate comprehensive test suites for web applications.

## Testing Workflow

### 1. Analysis
- Identify components to test
- Determine test types needed
- Plan test data requirements

### 2. Test Creation
- Write unit tests for isolated functions
- Create integration tests for connected components
- Develop end-to-end tests for user workflows

### 3. Quality Assurance
- Achieve >80% code coverage
- Test edge cases and error conditions
- Verify test independence

## Best Practices
- Use descriptive test names
- Follow Arrange-Act-Assert pattern
- Mock external dependencies
- Test one concept per test
```

### Mistake 7: Ignoring Context Limits

**Bad:**
```yaml
---
name: massive-skill
description: Does everything
---

[6,000 lines of instructions...]
```

**Why it's bad:** Exceeds the 5,000 word guideline for SKILL.md body, wasting tokens.

**Good:**
```yaml
---
name: comprehensive-skill
description: Multi-capability skill for [specific domain]
---

[Core instructions: 500 lines]

For detailed references:
- Complete API docs: `references/api-guide.md`
- Advanced patterns: `references/advanced-usage.md`
- All examples: `examples/` directory
```

## Quality Checklist

Before publishing your skill, verify:

### Content Quality
- [ ] YAML frontmatter is complete (name and description required)
- [ ] Skill name matches directory name (hyphen-case)
- [ ] Description clearly explains when to activate the skill
- [ ] SKILL.md body is under 5,000 words
- [ ] Instructions use imperative voice
- [ ] Workflow steps are clear and actionable
- [ ] Examples are relevant and correct

### Structure Quality
- [ ] Appropriate use of progressive disclosure
- [ ] References to external files work correctly
- [ ] Helper scripts are documented
- [ ] Examples are in separate files if >2 inline examples
- [ ] Directory structure follows conventions

### Technical Quality
- [ ] All file paths are correct
- [ ] Scripts are executable and documented
- [ ] Code examples are syntactically correct
- [ ] References link to existing files
- [ ] No broken internal links

### Usability Quality
- [ ] Clear activation conditions
- [ ] Logical workflow progression
- [ ] Appropriate level of detail
- [ ] Helpful error guidance
- [ ] Quality standards are explicit

### Testing Quality
- [ ] Skill activates with expected triggers
- [ ] Workflow executes successfully
- [ ] Scripts run without errors
- [ ] Examples work as documented
- [ ] Edge cases are handled

### Style Quality
- [ ] Professional tone throughout
- [ ] Consistent formatting
- [ ] Proper heading hierarchy
- [ ] Clear section organization
- [ ] No excessive decoration

## Real Examples Analysis

### Example 1: internal-comms (Simple Multi-Use Skill)

**Stats:** ~45-50 lines, modular structure

**Structure:**
```yaml
---
name: internal-comms
description: Draft clear, actionable internal communications including Slack messages, emails, and announcements
---

# Internal Communications

## Purpose
[Brief overview]

## Usage Mode 1: Slack Messages
[Instructions for Slack]

## Usage Mode 2: Email
[Instructions for email]

## Usage Mode 3: Announcements
[Instructions for announcements]

## General Principles
- Clarity over cleverness
- Actionable over informational
- Concise over comprehensive
```

**Why it works:**
- Clear modular sections for different use cases
- Each mode is self-contained
- General principles apply across all modes
- Concise enough to load quickly
- Activation description mentions all three modes

**Key Takeaways:**
- Multiple capabilities can coexist in one skill
- Keep each mode brief and focused
- Share common principles at the end
- Simple skills can still be powerful

### Example 2: webapp-testing (Medium Technical Skill)

**Stats:** ~110 lines, progressive complexity

**Structure:**
```markdown
---
name: webapp-testing
description: Comprehensive web application testing across functionality, UI/UX, performance, security, and accessibility
---

# Web Application Testing

## Purpose and Scope
[Clear overview of testing approach]

## Pre-Testing Analysis
[Quick codebase review steps]

## Core Testing Workflow

### 1. Functionality Testing
[Detailed steps]

### 2. UI/UX Testing
[Detailed steps]

### 3. Performance Testing
[Detailed steps]

### 4. Security Testing
[Detailed steps]

### 5. Accessibility Testing
[Detailed steps]

## Test Documentation
[How to document findings]

## Quality Standards
[Completion criteria]
```

**Why it works:**
- Structured workflow that's easy to follow
- Progressive through different testing types
- Each section has concrete steps
- Clear quality standards at the end
- Balances comprehensiveness with brevity

**Key Takeaways:**
- Medium skills can cover complex topics
- Clear phase structure helps users navigate
- Each subsection should be actionable
- End with clear completion criteria

### Example 3: mcp-builder (Complex Technical Skill)

**Stats:** ~550 lines, workflow-centric architecture

**Structure:**
```markdown
---
name: mcp-builder
description: Build Model Context Protocol servers with proper architecture, resource/tool/prompt implementations
---

# MCP Server Builder

## Philosophy
[Technical overview and principles]

## Architecture Overview
[Server structure explanation]

## Phase 1: Server Bootstrap
### 1.1 Project Setup
[Detailed steps]

### 1.2 Configuration
[Detailed steps]

## Phase 2: Core Implementation
### 2.1 Resource Implementation
[Detailed steps with code examples]

### 2.2 Tool Implementation
[Detailed steps with code examples]

### 2.3 Prompt Implementation
[Detailed steps with code examples]

## Phase 3: Testing and Validation
### 3.1 Unit Tests
[Testing approach]

### 3.2 Integration Tests
[Testing approach]

### 3.3 Manual Testing
[Testing steps]

## Phase 4: Documentation
[Documentation requirements]

## Best Practices
[Technical best practices]

## Common Patterns
[Reusable code patterns]

## Troubleshooting
[Common issues and solutions]
```

**Why it works:**
- Clear phase-based progression
- Technical depth where needed
- Code examples inline (appropriate for this skill)
- Comprehensive but organized
- Each phase has clear deliverables

**Key Takeaways:**
- Complex skills benefit from phase structure
- Technical skills can include more code examples
- Deep nesting (H4) is acceptable when needed
- Philosophy section helps orient users
- Troubleshooting section prevents common issues

**Progressive Disclosure Opportunity:**
If this skill grew beyond 650 lines, consider moving:
- Complete API reference to `references/mcp-api.md`
- More code examples to `examples/patterns/`
- Testing utilities to `scripts/test_server.py`

### Comparison: Length vs Complexity

| Skill | Lines | Complexity | Pattern |
|-------|-------|------------|---------|
| internal-comms | 50 | Low | Modular multi-use |
| webapp-testing | 110 | Medium | Progressive complexity |
| mcp-builder | 550 | High | Workflow-centric |

**Insight:** Line count should reflect actual complexity and workflow depth, not arbitrary padding. Each skill uses exactly the length needed to convey its instructions clearly.

## Appendix: Quick Reference

### File Name Conventions
- Skill directory: `hyphen-case-name/`
- Main file: `SKILL.md` (must be this exact name)
- Scripts: `descriptive-name.py` or `descriptive-name.sh`
- References: `descriptive-name.md`
- Examples: `descriptive-example-name.md`

### YAML Frontmatter Template
```yaml
---
name: your-skill-name
description: Clear description of what it does and when to use it
version: 1.0.0
license: MIT
allowed-tools: [Read, Write, Edit, Bash]
---
```

### Minimum Viable Skill
```markdown
---
name: minimal-skill
description: Does exactly one thing very well. Use when you need that specific thing.
---

# Minimal Skill

## Purpose
One-sentence explanation.

## Instructions
1. Step one
2. Step two
3. Step three

## Quality Check
- Criterion 1
- Criterion 2
```

### Length Guidelines at a Glance
- Simple: 50-120 lines
- Medium: 120-350 lines
- Complex: 350-650 lines
- If >650 lines: Use progressive disclosure

### Progressive Disclosure Decision Tree
```
Is this content >100 lines?
├─ No → Keep in SKILL.md
└─ Yes → Is it core workflow?
    ├─ Yes → Keep in SKILL.md (up to 650 lines total)
    └─ No → Is it documentation/reference?
        ├─ Yes → Move to references/
        ├─ No → Is it executable code?
        │   ├─ Yes → Move to scripts/
        │   └─ No → Is it examples?
        │       ├─ Yes → Move to examples/
        │       └─ No → Consider if it's needed
```

### Common Section Headers
```markdown
# Skill Name (H1 - only once)

## Purpose / Overview / Philosophy
## When to Use This Skill
## Core Workflow / Instructions / Process
## Key Principles / Best Practices
## Common Patterns / Examples
## Quality Standards / Completion Criteria
## Troubleshooting / Common Issues
## Additional Resources / References
```

### Testing Command Template
```bash
#!/bin/bash
# Test skill structure and references

echo "Testing skill: $1"

# Check required files
test -f "$1/SKILL.md" || echo "ERROR: SKILL.md missing"

# Validate YAML frontmatter
head -n 10 "$1/SKILL.md" | grep -q "^name:" || echo "ERROR: name field missing"
head -n 10 "$1/SKILL.md" | grep -q "^description:" || echo "ERROR: description field missing"

# Check file references
grep -o 'references/[^)]*' "$1/SKILL.md" | while read ref; do
    test -f "$1/$ref" || echo "WARNING: Referenced file not found: $ref"
done

# Check line count
lines=$(wc -l < "$1/SKILL.md")
echo "SKILL.md line count: $lines"
if [ $lines -gt 650 ]; then
    echo "RECOMMENDATION: Consider progressive disclosure (>650 lines)"
fi

echo "Testing complete"
```

---

## Conclusion

Creating effective Claude Skills is about clarity, structure, and appropriate detail. Remember:

1. **Start with a clear description** - Claude needs to know when to activate your skill
2. **Keep it focused** - Skills should have clear, specific purposes
3. **Use progressive disclosure** - Keep SKILL.md under 5,000 words, move details to references
4. **Write imperatively** - Give direct instructions, not suggestions
5. **Test thoroughly** - Verify activation, workflow, and all resources

The best skills are those that guide Claude through complex tasks with clarity and precision, providing exactly the right amount of detail at the right time.

For more examples and patterns, study the official Anthropic skills repository. Your skill should feel consistent with those professional examples while serving your specific use case.

Happy skill building!
