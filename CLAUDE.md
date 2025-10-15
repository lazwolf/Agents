# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Vantagepoint Marketplace repository containing Claude Code plugins for Vantagepoint development. The marketplace provides specialized agents for SQL script generation, UI configuration, and database management.

## Project Structure

```
/
├── .claude-plugin/
│   └── marketplace.json          # Marketplace manifest defining available plugins
├── vantagepoint-tools/          # Main plugin package
│   ├── .claude-plugin/
│   │   └── plugin.json         # Plugin configuration
│   └── agents/                 # Specialized agents
│       ├── screen-designer.md  # SQL script generation for UI configuration
│       ├── rollback.md         # Rollback script generator
│       ├── validator.md        # Configuration validator
│       └── workflow-coordinator.md # Multi-step workflow orchestrator
```

## Development Commands

### Agent Invocation

The Vantagepoint agents are invoked through Claude using natural language requests, not direct CLI commands:

```
# Example requests to Claude:
"Add a CustomerType field to the Invoice screen"
"Validate the Projects hub configuration"
"Generate a rollback script for recent changes"
"Use workflow coordinator to add multiple fields with validation"
```

Claude will invoke the appropriate agent(s) using the Task tool with the correct subagent_type parameter.

### Git Workflow

```bash
# Stage changes
git add .

# Commit with descriptive message
git commit -m "Description of changes"

# Push to remote
git push origin main
```

## Architecture and Agent System

### Agent Architecture

The plugin uses a specialized agent system where each agent has specific responsibilities:

1. **screen-designer**: Generates complete SQL script packages (01-11 numbered scripts)
   - Creates table DDL with proper UDIC structure
   - Handles FW_CustomColumnsData table operations with TabID management
   - Manages SEField entries for all required roles
   - Creates UDIC entity configurations with tab setup
   - Generates audit triggers (Insert/Update/Delete)
   - Supports dropdown fields via FW_CustomColumnValuesData
   - Configures custom grids with FW_CustomGridsData registration
   - Optional workflow configuration support

2. **rollback**: Creates safe rollback scripts with dependency checking
   - Generates dependency-aware deletion scripts
   - Creates backup tables before data removal
   - Follows proper deletion order (child before parent)
   - Preserves audit trails
   - Supports dry-run mode for preview

3. **validator**: Validates configurations with detailed reporting
   - Verifies all 6 required components per field
   - Checks GUID format (no hyphens)
   - Validates TabID consistency across tables
   - Verifies permissions (SEField and SEInfoCenter)
   - Identifies missing components
   - Supports dry-run mode for validation planning

4. **workflow-coordinator**: Orchestrates multi-agent workflows
   - Sequences validator, designer, and rollback agents
   - Manages state between agent calls
   - Implements checkpoint and recovery
   - Supports parallel validation operations
   - Creates deployment packages

### Key Technical Requirements

#### GUID Generation
Always use GUIDs without hyphens for Vantagepoint:
```sql
REPLACE(CAST(NEWID() AS VARCHAR(36)), '-', '')
```

#### TabID Consistency
Ensure TabID values match across:
- FW_CustomColumnsData
- FW_TabConfig
- SEFields
- Related configuration tables

#### Required Database Tables
Agents interact with:
- FW_CustomColumnsData (UI field configuration)
- SEFields (permission management)
- UDICEntity tables (custom entity data)
- FW_TabConfig (screen tab configuration)

## Plugin Development

### Adding New Agents

1. Create agent definition in `vantagepoint-tools/agents/[agent-name].md`
2. Update `vantagepoint-tools/.claude-plugin/plugin.json` with agent metadata
3. Test locally by asking Claude to invoke the new agent with a test task

### Marketplace Updates

1. Modify `.claude-plugin/marketplace.json` to add/update plugins
2. Update version numbers in both marketplace.json and plugin.json
3. Test installation from local marketplace before publishing

## Script File Naming Convention

The screen-designer agent generates numbered SQL scripts following this pattern:

| Number | Script Type | Purpose | Auditing |
|--------|------------|---------|----------|
| 01 | Create | Table DDL creation | No |
| 02 | Registration | UDIC registration | Yes |
| 03 | Triggers | Audit triggers | No |
| 04 | Fields | FW_CustomColumnsData | Yes |
| 05 | Captions | Captions and labels | Yes |
| 06 | ScreenLayout | CFGScreenDesignerData | Yes |
| 07 | Security | SEInfoCenter/SEField | Yes |
| 08 | SampleData | Test data | Yes |
| 09 | Verify | Validation queries | No |
| 10 | Workflow | Optional workflows | Yes |
| 11 | DefaultData | Production data | Yes |
| 99 | Rollback | Cleanup script | Yes |

Format: `[Number]_[EntityName]_[ScriptType].sql`
Example: `06_ProjectDashboard_ScreenLayout.sql`

## Testing Workflow

1. **Pre-deployment Testing**:
   Ask Claude to perform test operations:
   - "Check test configuration using validator"
   - "Add a test field to verify script generation"
   - "Generate rollback for the test field"

2. **Validation Sequence**:
   - Always run validator before changes
   - Generate scripts with screen-designer
   - Create rollback scripts immediately after changes
   - Use workflow-coordinator for complex multi-step operations

## Important Conventions

### SQL Performance Rules
- Use UNPIVOT operator for converting columns to rows (never CROSS APPLY with VALUES)
- Use PIVOT operator for converting rows to columns (never CASE statements with GROUP BY)
- Always use CTEs (WITH clause) to prepare data before PIVOT/UNPIVOT operations

### Vantagepoint Naming Conventions
- "Vantagepoint" has no capital P (not "VantagePoint")
- Can be abbreviated as "Vp" in prompts only
- API endpoints are never plural
- API responses don't use a "data" key

### Agent Invocation Patterns
- Agents receive SQL Server connection via MCP tools
- Each agent operates independently (stateless)
- Workflow coordinator manages multi-agent sequences
- All agents generate scripts with validation checks

## Common Tasks

### Adding Custom Fields
Ask Claude: "Add CustomerType field to Invoice screen with dropdown values: Regular, Premium, VIP"

### Creating UDIC Entities
Ask Claude: "Create UDIC entity CustomerExtensions with fields: Type (varchar), Category (int), Region (varchar)"

### Validating Changes
Ask Claude: "Validate all custom fields on Invoice screen and check permissions"

### Generating Rollbacks
Ask Claude: "Generate rollback script for CustomerType field on Invoice screen"

### Complex Workflows
Ask Claude: "Add 5 custom fields to Project screen with full validation and rollback generation using workflow coordinator"