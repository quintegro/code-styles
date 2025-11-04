# 🎨 Code Styles

> **Purpose:** Centralized repository for maintaining compressed and comprehensive versions of code style documents from Confluence. This serves as a single source of truth for development standards across all programming languages and frameworks used in our organization.

## 📋 Overview

This repository contains standardized, AI-ready code style guidelines that have been extracted and normalized from our internal Confluence documentation. Each language-specific directory contains comprehensive style guides optimized for:

- **LLM-based Code Review** - Clear, unambiguous rules with stable anchors
- **Developer Reference** - Quick lookup for style decisions and patterns
- **Automated Tooling** - Integration with linters, formatters, and CI/CD pipelines
- **Team Consistency** - Single source of truth to eliminate style debates

## 🗂️ Repository Structure

```txt
code-styles/
├── README.md               # This file - repository overview and guidelines
├── go/                     # Go language standards
│   └── README.md           # Go Engineering Standards & Code Style (AI-Ready)
├── [future-languages]/     # Additional language directories as needed
```

### Current Language Support

| Language | Status    | Version | Last Updated | Maintainer       |
|----------|-----------|---------|--------------|------------------|
| **Go**   | Active    | 1.0     | 2025-10-07   | Internal Go Team |

## 🚀 Quick Start

### For Code Reviewers

1. Use rule anchors in review comments for consistency
2. Reference specific rules using their anchors (e.g., `§ERR-003` for error naming)
3. Ensure all team members have access to this repository

## 📖 Usage Guidelines

### Referencing Rules in Code Reviews

When commenting on code, reference specific rules using their anchors:

```markdown
❌ "This error handling looks wrong"
✅ "See §ERR-002 for proper error wrapping with %w"
```

General/AI Code Review should reference specific version of the rule when commenting on code.

### Automated Tooling

These style guides are designed to work with:

- **Linters**: `golangci-lint`, `eslint`, `pylint`, etc. are ajusted to code styles in this repository
- **Formatters**: `gofmt`, `prettier`, etc. are ajusted to code styles in this repository
- **CI/CD**: Automated style checking in pull requests
- **AI**: References this repository as a source of truth for making a code review

## 🔄 Maintenance & Updates

### Adding New Languages

1. Create new directory: `mkdir <language>/`
2. Add comprehensive README.md following the Go template structure
3. Update this main README.md with language entry
4. Ensure all rules have stable anchors and clear examples

### Updating Existing Guidelines

1. **Source**: Update originates from Confluence documentation
2. **Review**: TechLead reviews changes for accuracy
3. **Version**: Increment version number and update date
4. **Communication**: Notify teams of significant changes

### Version Control

- **Major versions** (1.0 → 2.0): Breaking changes or major restructuring
- **Minor versions** (1.0 → 1.1): New rules or significant clarifications
- **Patch versions** (1.0.0 → 1.0.1): Bug fixes and minor improvements

## 🤝 Contributing

### Process for Updates

1. **Identify Need**: Style question or inconsistency arises
2. **Research**: Check Confluence for authoritative source
3. **Draft**: Create comprehensive, unambiguous rule
4. **Review**: TechLead and team review
5. **Merge**: Update repository with proper versioning
6. **Communicate**: Notify affected teams

### Quality Standards

All style guides must include:

- ✅ **Clear, unambiguous rules** - No room for interpretation
- ✅ **Stable anchors** - For consistent referencing
- ✅ **Code examples** - Both good and bad patterns
- ✅ **Rationale** - Why the rule exists
- ✅ **AI-ready format** - Optimized for LLM understanding

## 📚 Additional Resources

### External References

- **General**: [Clean Code](https://www.oreilly.com/library/view/clean-code/9780136083238/) principles
- **Go**: [Effective Go](https://golang.org/doc/effective_go.html)
