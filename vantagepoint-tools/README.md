# Vantagepoint Tools Plugin

A comprehensive Claude Code plugin providing production-ready SQL script generation for Vantagepoint UI configuration, database management, and workflow automation.

## Overview

This plugin provides four specialized agents that work together to generate syntactically correct SQL scripts while understanding Vantagepoint's complex table relationships and avoiding common configuration pitfalls:

- **Screen Designer**: Generates complete SQL script packages (01-11 numbered scripts) for UI configuration
- **Rollback**: Creates safe rollback scripts with dependency checking and backup table creation
- **Validator**: Validates configurations, identifies missing components, and verifies permissions
- **Workflow Coordinator**: Orchestrates multi-agent workflows for complex operations

## How to Use

These agents are invoked through Claude Code, not as direct CLI commands. Simply describe what you want to accomplish, and Claude will use the appropriate agent.

### Agent Invocation

**Instead of typing commands**, you ask Claude in natural language:
- "Add a CustomerType field to the Invoice screen"
- "Validate the Projects hub configuration"
- "Create a rollback script for recent changes"
- "Create a new UDIC entity for project tracking"

Claude will then invoke the appropriate agent(s) to complete your task.

## Available Agents

### 1. Screen Designer Agent (`screen-designer`)

Generates complete SQL script packages for Vantagepoint UI configuration, following Vantagepoint naming conventions and best practices.

**Usage:**
```
Ask Claude: "Add a custom field called 'CustomerType' to the Invoice screen"
```

**Script Generation Capabilities:**
- **01_Create.sql**: Table DDL with proper structure (UDIC_UID as primary key)
- **02_Registration.sql**: UDIC registration in FW_UDICData, FW_UDICLocalized, FW_CFGLabelData
- **03_Triggers.sql**: Three audit triggers per table (Insert/Update/Delete)
- **04_Fields.sql**: Field definitions in FW_CustomColumnsData with proper TabID
- **05_Captions.sql**: Captions in FW_CustomColumnCaptions and CFGScreenDesignerLabels
- **06_ScreenLayout.sql**: Screen positioning in CFGScreenDesignerData (27 columns)
- **07_Security.sql**: SEInfoCenter and SEField entries for all roles
- **08_SampleData.sql**: Test data for development
- **09_Verify.sql**: Validation queries
- **10_Workflow.sql**: Optional workflow configuration
- **11_DefaultData.sql**: Production reference data
- **99_Rollback.sql**: Complete cleanup script

**Key Features:**
- Generates GUIDs without hyphens: `REPLACE(CAST(NEWID() AS VARCHAR(36)), '-', '')`
- Validates TabID consistency across tables
- Handles dropdown fields with FW_CustomColumnValuesData
- Supports custom grids with FW_CustomGridsData registration
- Wraps DML in proper auditing context
- Uses INSERT WHERE NOT EXISTS pattern for upgrade safety

### 2. Rollback Agent (`rollback`)

Generates safe rollback scripts that properly handle dependencies and preserve data integrity.

**Usage:**
```
Ask Claude: "Generate rollback script for the CustomerType field addition"
```

**Capabilities:**
- **Dependency Analysis**: Checks for field usage before deletion
- **Backup Table Creation**: Creates _Backup tables before data removal
- **Proper Deletion Order**: Child records before parent (workflow cleanup first)
- **Configuration Cleanup**: Removes from all 6 required component tables
- **Audit Trail Preservation**: Maintains audit history during rollback
- **Dry-Run Mode**: Preview rollback operations without execution

**Safety Features:**
- IF EXISTS checks for all deletions
- Transaction wrapping with error handling
- Warnings about potential data loss
- Verification queries before rollback
- Conservative approach when MCP unavailable

### 3. Validator Agent (`validator`)

Comprehensive validation of Vantagepoint UI configurations with detailed reporting.

**Usage:**
```
Ask Claude: "Check if the CustomerType field is properly configured"
Or: "dry-run: Show validation plan for UDIC_ProjectDashboard"
```

**Validation Checks:**
- **6-Component Verification**: Physical column, FW_CustomColumnsData, Caption, ScreenLayout, Label, SEField
- **GUID Format**: Ensures no hyphens in GUIDs
- **TabID Consistency**: Verifies alignment across configuration tables
- **Permission Validation**: Checks SEField and SEInfoCenter for all required roles
- **Grid Registration**: Verifies FW_CustomGridsData entries for custom grids
- **Dropdown Configuration**: Validates FW_CustomColumnValuesData for LimitToList fields
- **Workflow Integrity**: Checks event/condition/action relationships

**Reporting Features:**
- Component status checklist with ✓/✗ indicators
- Critical issues highlighted
- Resolution recommendations
- SQL fix scripts generated
- Dry-run mode for validation planning

### 4. Workflow Coordinator Agent (`workflow-coordinator`)

Orchestrates complex multi-step UI configuration workflows using predefined patterns.

**Usage:**
```
Ask Claude: "Add multiple custom fields to Invoice screen with full validation"
```

**Workflow Patterns:**
- **Standard Field Addition**: Validate → Check Exists → Generate → Rollback → Package
- **Bulk Field Operations**: Parallel validation → Bulk script generation → Transaction planning
- **UDIC Entity Creation**: Name validation → Table creation → Registration → Field setup → Security
- **Complex Grid Setup**: Parent entity → Grid table → Registration → Field configuration → UI placement

**Orchestration Features:**
- **Multi-Agent Coordination**: Sequences validator, designer, and rollback agents
- **State Management**: Maintains context across operations
- **Error Recovery**: Automatic retry with exponential backoff
- **Checkpoint System**: Resume from last successful step
- **Deployment Package**: Creates numbered script packages ready for execution
- **Parallel Processing**: Runs independent validations concurrently
- **Transaction Planning**: Groups related operations for atomic execution

## Common Workflows

### Adding a Custom Field

1. **Simple Addition:**
   ```
   Ask Claude: "Add 'Priority' field to Project screen"
   ```

2. **With Validation:**
   ```
   Ask Claude: "Add 'Priority' field with validation and rollback generation"
   ```

### Creating UDIC Entity

```
Ask Claude: "Create UDIC entity for CustomerExtensions with fields: Type, Category, Region"
```

### Validating Configuration

```
Ask Claude: "Validate all custom fields on Invoice screen"
```

### Emergency Rollback

```
Ask Claude: "Generate and execute rollback for today's Invoice screen changes"
```

## Best Practices

### 1. Always Validate First
Before making changes, run validation to check current state:
```
Ask Claude: "Check Invoice screen configuration"
```

### 2. Use Workflow Coordinator for Complex Operations
For multi-step operations, use the coordinator:
```
Ask Claude: "Complete Invoice screen customization with 5 new fields"
```

### 3. Generate Rollback Scripts
Always generate rollback scripts after changes:
```
Ask Claude: "Generate rollback for all changes made today"
```

### 4. Test in Non-Production
Test all scripts in development environment first.

## Configuration Requirements

### Database Access
Agents require access to Vantagepoint MS SQL Server database with specific tables:

#### Core Configuration Tables
- **FW_CustomColumnsData** (25 columns, TabID at position 5)
- **CFGScreenDesignerData** (27 columns, TabID at position 2)
- **CFGScreenDesignerLabels** (7 columns, no TabID)
- **FW_CustomColumnCaptions** (5 columns)
- **FW_CustomColumnValuesData** (11 columns - actual table, NOT the view)
- **FW_CustomColumns** (27 columns, PropertyBag support)

#### UDIC Management Tables
- **FW_UDICData** (9 columns - registration table)
- **FW_UDICLocalized** (3 columns - localization)
- **FW_CFGLabelData** (7 columns, all NOT NULL)
- **FW_InfoCenterTabsData** (9 columns - tab configuration)
- **FW_InfoCenterTabHeadings** (5 columns - tab labels)
- **FW_CustomGridsData** (9 columns - grid registration)
- **FW_CustomGridCaptions** (4 columns - grid labels)

#### Security Tables
- **SEInfoCenter** (11 columns - hub security)
- **SEField** (11 columns - field security, REQUIRED for access)
- **SE** (role definitions)

#### Workflow Tables (Optional)
- **WorkflowEvents** (10 columns)
- **WorkflowConditions** (10 columns, ID links to EventID)
- **WorkflowActions** (9 columns)
- **WorkflowActionColumnUpdate** (10 columns, uses NewValue not UpdateValue)
- **WorkflowActionSproc** (4 columns)

### MCP MSSQL Server Setup
Configure the MCP MSSQL server connection:

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

## Important Technical Notes

### GUID Generation
Always generate GUIDs without hyphens:
```sql
REPLACE(CAST(NEWID() AS VARCHAR(36)), '-', '')
```

### Table Name Constraints
- **Maximum Length**: 32 characters (FW_UDICData.UDIC_ID is VARCHAR(32))
- **UDIC Pattern**: UDIC_EntityName (entity < 27 chars)
- **Grid Pattern**: UDIC_HubName_GridName (total ≤32 chars)

### TabID Requirements
Tables WITH TabID:
- FW_CustomColumnsData (position 5)
- CFGScreenDesignerData (position 2)
- FW_InfoCenterTabsData

Tables WITHOUT TabID:
- CFGScreenDesignerLabels
- FW_CustomColumnCaptions
- SEField
- SEInfoCenter

### Field Configuration Requirements
Every custom field requires 6 components:
1. Physical column in table
2. FW_CustomColumnsData entry (field definition)
3. FW_CustomColumnCaptions entry (display caption)
4. CFGScreenDesignerData entry (screen positioning)
5. CFGScreenDesignerLabels entry (field label)
6. SEField entry (security - REQUIRED for access)

### Dropdown Field Configuration
Fields with predefined values:
- Set DataType='dropdown' (NOT 'string')
- Set LimitToList='Y'
- Add values to FW_CustomColumnValuesData (NOT the view)
- Include UICultureName, DataValue, and Seq columns

### Grid Configuration
Custom grids require:
- Grid table with composite key (UDIC_UID, Seq)
- FW_CustomGridsData registration entry
- FW_CustomGridCaptions for display name
- Individual CFGScreenDesignerData entries for each grid field
- Grid component entry with ComponentType='component_grid'

### Security Configuration
- Grant access to all roles with AccessAllNavNodes='Y' or Role='DEFAULT'
- Use INSERT WHERE NOT EXISTS pattern to avoid duplicates
- System fields in SEField must have appropriate ReadOnly settings
- Dividers have NULL for Tablename and ColumnName

### Auditing Requirements
- DML scripts require auditing wrapper
- DDL scripts (CREATE TABLE, triggers) do NOT use auditing wrapper
- Insert triggers audit PRIMARY KEY only
- Update triggers audit ALL fields with IF UPDATE() checks
- Delete triggers audit the primary key

## Troubleshooting

### Field Not Appearing
1. Run validator to check configuration
2. Verify permissions
3. Check TabID alignment

### Script Execution Errors
1. Check database connectivity
2. Verify table existence
3. Review error logs

### Rollback Issues
1. Check backup table existence
2. Verify no active dependencies
3. Review transaction logs

## Examples

### Complete Field Addition Workflow
```
# 1. Validate current state
Ask Claude: "Check Project screen configuration"

# 2. Add field with workflow coordinator
Ask Claude: "Add Priority field to Project screen using workflow coordinator"

# 3. Claude will orchestrate the workflow to:
#    - Pre-validate configuration
#    - Generate scripts
#    - Execute changes
#    - Post-validate results
#    - Generate rollback scripts
```

### Bulk Field Addition
```
Ask Claude: "Add fields to Invoice: CustomerType (dropdown), Priority (int), Region (varchar), Notes (text)"
```

### Configuration Audit
```
Ask Claude: "Perform complete audit of all custom fields and UDIC entities"
```

## Support

For issues or questions:
1. Check validation reports for detailed error information
2. Review generated SQL scripts for syntax issues
3. Consult Vantagepoint documentation for table structures
4. Contact plugin maintainer

## Version History

### v1.0.0
- Initial release
- Four core agents: screen-designer, rollback, validator, workflow-coordinator
- Full CRUD operations for custom fields
- UDIC entity management
- Comprehensive validation system

## License

MIT License - See LICENSE file for details

## Contributing

Contributions welcome! Please:
1. Follow existing patterns
2. Add tests for new features
3. Update documentation
4. Submit pull requests

## Acknowledgments

Built for the Vantagepoint development community to streamline UI configuration and reduce common errors.