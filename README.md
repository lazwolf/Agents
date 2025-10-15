# Vantagepoint Marketplace

A comprehensive Claude Code plugin marketplace providing specialized agents for Vantagepoint database development, UI configuration, and SQL script generation.

## Overview

This marketplace provides production-ready tools for Vantagepoint developers, featuring intelligent agents that generate syntactically correct SQL scripts, validate configurations, and orchestrate complex workflows while avoiding common pitfalls.

## Quick Start

### Prerequisites

- Claude Code CLI installed and configured
- Access to Vantagepoint MS SQL Server database
- MCP MSSQL server configured (for database operations)

### How to Use

These agents are available through Claude Code. Simply describe what you want to accomplish in natural language, and Claude will invoke the appropriate agent(s) for you.

**Note:** The agents are not invoked directly via CLI commands. Instead, you interact with them through Claude by describing your task.

Example interaction:
```
You: "Add a CustomerType field to the Invoice screen"
Claude: [Invokes screen-designer agent to generate SQL scripts]
```

### Available Plugins

#### Vantagepoint Tools
A comprehensive toolkit containing four specialized agents that work together:

- **screen-designer**: Generates complete SQL script packages for UI configuration
- **rollback**: Creates safe rollback scripts with dependency checking
- **validator**: Validates configurations and identifies missing components
- **workflow-coordinator**: Orchestrates multi-agent workflows for complex operations

## Marketplace Structure

```
/
├── .claude-plugin/
│   └── marketplace.json               # Marketplace manifest
├── vantagepoint-tools/                # Plugin package
│   ├── .claude-plugin/
│   │   └── plugin.json                # Plugin configuration
│   ├── agents/
│   │   ├── screen-designer.md        # Screen designer agent
│   │   ├── rollback.md               # Rollback generator
│   │   ├── validator.md              # Configuration validator
│   │   └── workflow-coordinator.md   # Workflow orchestrator
│   └── README.md                      # Plugin documentation
└── README.md                          # This file
```

## Plugin: Vantagepoint Tools

### Overview
Production-ready SQL script generation for Vantagepoint UI configuration, featuring intelligent agents that understand Vantagepoint's complex table relationships and requirements.

### Key Features
- **Complete SQL Script Packages**: Generates numbered scripts (01-11) following Vantagepoint conventions
- **Automated GUID Generation**: Properly formatted GUIDs without hyphens using `REPLACE(CAST(NEWID() AS VARCHAR(36)), '-', '')`
- **TabID Validation**: Ensures consistency across FW_CustomColumnsData, FW_TabConfig, and CFGScreenDesignerData
- **Dependency Management**: Checks dependencies before modifications, proper deletion order
- **Transaction Safety**: All DML scripts wrapped in proper auditing and transaction handling
- **Workflow Automation**: Orchestrates multi-step operations with checkpoint/recovery

### Agent Capabilities

#### Screen Designer
- **Script Generation**: Creates complete SQL script packages (01_Create.sql through 11_DefaultData.sql)
- **Custom Field Creation**: Handles all 6 required components (physical column, FW_CustomColumnsData, captions, screen layout, labels, security)
- **UDIC Entity Management**: Complete entity creation with tables, triggers, registration, and tab configuration
- **Grid Configuration**: Supports custom grids with proper FW_CustomGridsData registration
- **Dropdown Fields**: Configures fields with predefined values using FW_CustomColumnValuesData
- **Workflow Support**: Generates workflow events, conditions, and actions

#### Rollback Generator
- **Safe Deletion Scripts**: Generates scripts with proper dependency checking
- **Backup Table Creation**: Creates backup tables before data removal
- **Deletion Order Management**: Follows correct sequence (child records before parent)
- **Audit Trail Preservation**: Maintains audit history during rollback
- **Dry-Run Support**: Previews rollback operations without execution

#### Validator
- **Component Verification**: Checks all 6 required components for field configuration
- **GUID Format Validation**: Ensures GUIDs are properly formatted without hyphens
- **TabID Consistency**: Verifies TabID alignment across all configuration tables
- **Permission Validation**: Checks SEField and SEInfoCenter entries
- **Missing Component Detection**: Identifies gaps in configuration
- **Dry-Run Mode**: Shows validation plan without database access

#### Workflow Coordinator
- **Multi-Agent Orchestration**: Coordinates validator, designer, and rollback agents
- **Workflow Patterns**: Pre-configured patterns for common operations
- **State Management**: Maintains context across agent operations
- **Error Recovery**: Automatic retry and rollback on failure
- **Deployment Package Creation**: Assembles complete script packages for deployment

## Usage Examples

**Important:** These agents are invoked through Claude (me), not as direct CLI commands. Simply describe what you want to accomplish, and I'll use the appropriate agent.

### Add a Custom Field
```
# Ask Claude:
"Add a CustomerType dropdown field to Invoice screen with values: Regular, Premium, VIP"

# Or with complete specifications:
"Add CustPriority field to Projects hub, type=numeric, tab=general, position=row 5 column 2"
```

### Create UDIC Entity
```
# Ask Claude:
"Create UDIC_ProjectDashboard entity with fields: CustName (string), CustStatus (dropdown: Active, Inactive), CustBudget (currency), CustStartDate (date)"

# Or with custom grid:
"Create UDIC_TeamManager with main fields and CustMemberGrid containing: CustEmployeeID (employee), CustRole (string), CustAllocation (numeric)"
```

### Validate Configuration
```
# Ask Claude:
"Check if CustomerType field is properly configured on Invoice screen"

# Or comprehensive validation:
"Validate all custom fields on Projects hub including permissions and screen layout"

# Or dry-run validation:
"dry-run: Show validation plan for UDIC_ProjectDashboard"
```

### Complex Workflow
```
# Ask Claude:
"Add 5 custom fields to Invoice screen: CustomerType, Priority, Region, Notes, ApprovalStatus with full validation and rollback generation"

# Or complete UDIC setup:
"Create UDIC_ProjectMetrics entity with validation, fields, grids, and generate deployment package"
```

### Generate Rollback
```
# Ask Claude:
"Generate rollback script for CustomerType field on Invoice screen"

# Or for entire entity:
"Create complete rollback script for UDIC_ProjectDashboard entity including all fields and configuration"

# Or with backups:
"Generate rollback for today's changes with backup table creation"
```

## Development

### Adding New Plugins

1. Create a new plugin directory
2. Add `.claude-plugin/plugin.json`
3. Add your agents, commands, or hooks
4. Update `marketplace.json` to include your plugin
5. Test locally before publishing

### Testing the Agents

To test the agents, simply ask Claude to perform tasks:

1. **Test validation**: "Check the configuration of the Projects hub"
2. **Test script generation**: "Add a test field to the Invoice screen"
3. **Test rollback**: "Generate a rollback script for the test field"
4. **Test workflow**: "Use workflow coordinator to add multiple fields with validation"

## Script File Naming Convention

The screen-designer agent generates SQL scripts following this naming pattern:

**Format**: `[Number]_[EntityName]_[ScriptType].sql`

| Number | Script Type | Purpose | Auditing Wrapper |
|--------|------------|---------|------------------|
| 01 | Create | Table DDL creation | No |
| 02 | Registration | UDIC registration (FW_UDICData, labels, tabs) | Yes |
| 03 | Triggers | Audit triggers (Insert/Update/Delete) | No |
| 04 | Fields | Field definitions (FW_CustomColumnsData) | Yes |
| 05 | Captions | Field captions and labels | Yes |
| 06 | ScreenLayout | Screen positioning (CFGScreenDesignerData) | Yes |
| 07 | Security | SEInfoCenter and SEField configuration | Yes |
| 08 | SampleData | Test data for development | Yes |
| 09 | Verify | Validation queries | No |
| 10 | Workflow | Optional workflow configuration | Yes |
| 11 | DefaultData | Production default data | Yes |
| 99 | Rollback | Cleanup/rollback script | Yes |

Example: `01_ProjectDashboard_Create.sql`, `06_ProjectDashboard_ScreenLayout.sql`

## Best Practices

### For Users

#### Pre-Implementation
1. **Always validate first**: Run validator before making changes
2. **Check name lengths**: UDIC table names must be ≤32 characters
3. **Plan grid names carefully**: `UDIC_HubName_GridName` pattern
4. **Use dry-run mode**: Test complex operations without database access

#### During Implementation
1. **Use workflow coordinator for complex tasks**: Multi-field additions, UDIC creation
2. **Specify field types explicitly**: dropdown vs string, currency vs numeric
3. **Include all required values**: Dropdown options, default values, tab positions
4. **Follow naming conventions**: Cust prefix for all custom fields

#### Post-Implementation
1. **Generate rollback scripts immediately**: Create after each change
2. **Run validation**: Confirm all components are properly configured
3. **Test in development first**: Never run untested scripts in production
4. **Save script packages**: Keep versioned copies of all generated scripts

### For Plugin Developers

#### Agent Development
1. **Follow established patterns**: Use existing agents as templates
2. **Support dry-run mode**: Allow preview without database access
3. **Include comprehensive error handling**: Check for MCP availability
4. **Provide clear feedback**: Use descriptive error messages

#### Documentation
1. **Document all capabilities**: Include hidden features in README
2. **Provide realistic examples**: Show actual use cases
3. **Explain prerequisites**: Database access, MCP configuration
4. **Keep agent definitions updated**: Sync with documentation

## Requirements

### Database Access
Vantagepoint tools require access to MS SQL Server database with the following tables:

#### Core Configuration Tables
- **FW_CustomColumnsData** - Field definitions (includes TabID at position 5)
- **CFGScreenDesignerData** - Screen layout (27 columns, includes TabID at position 2)
- **CFGScreenDesignerLabels** - Component labels (7 columns, no TabID)
- **FW_CustomColumnCaptions** - Field captions (5 columns)
- **FW_CustomColumnValuesData** - Dropdown values (NOT FW_CustomColumnValues which is a view)

#### UDIC Tables
- **FW_UDICData** - UDIC registration (9 columns)
- **FW_UDICLocalized** - UDIC localization
- **FW_CFGLabelData** - Label definitions (7 columns, all NOT NULL)
- **FW_InfoCenterTabsData** - Tab configuration
- **FW_InfoCenterTabHeadings** - Tab headings
- **FW_CustomGridsData** - Grid registration (required for custom grids)
- **FW_CustomGridCaptions** - Grid display names

#### Security Tables
- **SEInfoCenter** - Hub-level security (11 columns)
- **SEField** - Field-level security (11 columns, required for field access)
- **SE** - Role definitions

#### Workflow Tables (Optional)
- **WorkflowEvents** - Event definitions (10 columns)
- **WorkflowConditions** - Condition logic (10 columns)
- **WorkflowActions** - Action definitions (9 columns)
- **WorkflowActionColumnUpdate** - Column update actions
- **WorkflowActionSproc** - Stored procedure actions

### MCP MSSQL Server Configuration
Configure the MCP MSSQL server for database operations:

```bash
claude mcp add -s local MSSQL node "C:\GIT\Microsoft\SQL-AI-samples\MssqlMcp\Node\dist\index.js" \
  -e SERVER_NAME=your_server \
  -e DATABASE_NAME=your_database \
  -e READONLY=false \
  -e TRUST_SERVER_CERTIFICATE=true \
  -e ENCRYPT=true \
  -e AUTHENTICATION=sql \
  -e USERNAME=your_username \
  -e PASSWORD=your_password
```

Required MCP tools:
- mcp__MSSQL__describe_table
- mcp__MSSQL__read_data
- mcp__MSSQL__list_table
- mcp__MSSQL__insert_data
- mcp__MSSQL__update_data
- mcp__MSSQL__create_table
- mcp__MSSQL__create_index
- mcp__MSSQL__drop_table

## Support

### Getting Help
1. Check agent documentation in `/vantagepoint-tools/agents/`
2. Review validation reports for detailed errors
3. Consult generated SQL scripts for syntax issues

### Reporting Issues
Please report issues with:
- Agent name and version
- Error messages
- Steps to reproduce
- Expected vs actual behavior

## Contributing

We welcome contributions! Please:
1. Follow existing patterns and conventions
2. Add tests for new features
3. Update documentation
4. Submit pull requests with clear descriptions

### Development Setup

```bash
# Clone the repository
git clone https://github.com/owner/vantagepoint-marketplace.git

# Create a new branch
git checkout -b feature/your-feature

# Make changes and test by asking Claude to invoke your agents

# Submit pull request
```

## Version History

### v1.0.0 (Current)
- Initial marketplace release
- Vantagepoint Tools plugin with 4 core agents
- Comprehensive documentation
- Local testing support

## Roadmap

### Planned Features
- Additional validation rules
- Performance optimization tools
- Batch operation support
- Integration with CI/CD pipelines
- Extended UDIC entity operations

### Future Plugins
- Vantagepoint API toolkit
- Database migration assistant
- Performance analyzer
- Security audit tools

## License

MIT License - See [LICENSE](LICENSE) file for details.

## Acknowledgments

Built for the Vantagepoint development community to improve productivity and reduce configuration errors.

## Links

- [Plugin Documentation](vantagepoint-tools/README.md)
- [Claude Code Docs](https://docs.claude.com/en/docs/claude-code/)
- [Issue Tracker](https://github.com/owner/vantagepoint-marketplace/issues)
