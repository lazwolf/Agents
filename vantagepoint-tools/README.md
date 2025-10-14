# Vantagepoint Tools Plugin

A comprehensive Claude Code plugin providing specialized agents for Vantagepoint UI configuration, database management, and workflow automation.

## Overview

This plugin provides four specialized agents that work together to streamline Vantagepoint development:

- **Screen Designer**: Generates SQL scripts for UI configuration
- **Rollback**: Creates safe rollback scripts for changes
- **Validator**: Validates configurations and identifies issues
- **Workflow Coordinator**: Orchestrates multi-step operations

## Installation

### Via Marketplace

```bash
# Add the marketplace
/plugin marketplace add owner/vantagepoint-marketplace

# Install the plugin
/plugin install vantagepoint-tools@vantagepoint-marketplace
```

### Direct Installation

```bash
/plugin install owner/repo
```

## Available Agents

### 1. Screen Designer Agent (`screen-designer`)

Generates SQL scripts for Vantagepoint UI configuration with expertise in avoiding common pitfalls.

**Usage:**
```bash
/task screen-designer "Add a custom field called 'CustomerType' to the Invoice screen"
```

**Capabilities:**
- Custom field creation with proper GUID generation
- UDIC entity management
- Screen layout modifications
- TabID mismatch prevention
- SEField entry management

**Key Features:**
- Automatically generates GUIDs without hyphens
- Validates TabID consistency
- Creates comprehensive scripts with validation

### 2. Rollback Agent (`rollback`)

Generates safe rollback scripts to undo UI changes.

**Usage:**
```bash
/task rollback "Generate rollback script for the CustomerType field addition"
```

**Capabilities:**
- Safe field removal with dependency checking
- UDIC entity cleanup
- Configuration restoration
- Backup data generation

**Key Features:**
- Checks for dependencies before deletion
- Creates backup tables
- Preserves audit trails

### 3. Validator Agent (`validator`)

Validates Vantagepoint UI configurations and identifies missing components.

**Usage:**
```bash
/task validator "Check if the CustomerType field is properly configured"
```

**Capabilities:**
- Configuration completeness verification
- Missing component identification
- Permission validation
- Cross-table consistency checks

**Key Features:**
- Comprehensive validation reports
- Error resolution recommendations
- Performance impact assessment

### 4. Workflow Coordinator Agent (`workflow-coordinator`)

Orchestrates complex multi-step UI configuration workflows.

**Usage:**
```bash
/task workflow-coordinator "Add multiple custom fields to Invoice screen with full validation"
```

**Capabilities:**
- Multi-agent coordination
- Sequential operation management
- Error recovery
- State management

**Key Features:**
- Automated workflow execution
- Progress tracking
- Checkpoint system
- Automatic rollback on failure

## Common Workflows

### Adding a Custom Field

1. **Simple Addition:**
   ```bash
   /task screen-designer "Add 'Priority' field to Project screen"
   ```

2. **With Validation:**
   ```bash
   /task workflow-coordinator "Add 'Priority' field with validation and rollback generation"
   ```

### Creating UDIC Entity

```bash
/task screen-designer "Create UDIC entity for CustomerExtensions with fields: Type, Category, Region"
```

### Validating Configuration

```bash
/task validator "Validate all custom fields on Invoice screen"
```

### Emergency Rollback

```bash
/task rollback "Generate and execute rollback for today's Invoice screen changes"
```

## Best Practices

### 1. Always Validate First
Before making changes, run validation to check current state:
```bash
/task validator "Check Invoice screen configuration"
```

### 2. Use Workflow Coordinator for Complex Operations
For multi-step operations, use the coordinator:
```bash
/task workflow-coordinator "Complete Invoice screen customization with 5 new fields"
```

### 3. Generate Rollback Scripts
Always generate rollback scripts after changes:
```bash
/task rollback "Generate rollback for all changes made today"
```

### 4. Test in Non-Production
Test all scripts in development environment first.

## Configuration Requirements

### Database Access
Agents require access to Vantagepoint database tables:
- FW_CustomColumnsData
- SEFields
- UDICEntity
- FW_TabConfig

### MCP Tools
The following MCP tools should be available:
- mcp__MSSQL__describe_table
- mcp__MSSQL__read_data
- mcp__MSSQL__insert_data
- mcp__MSSQL__update_data

## Important Notes

### GUID Generation
Always use this format for GUIDs in Vantagepoint:
```sql
REPLACE(CAST(NEWID() AS VARCHAR(36)), '-', '')
```

### TabID Consistency
Ensure TabIDs match across:
- FW_CustomColumnsData
- FW_TabConfig
- Related configuration tables

### Permission Management
All custom fields require:
- SEField entry
- Permission configuration
- Role assignments

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
```bash
# 1. Validate current state
/task validator "Check Project screen configuration"

# 2. Add field with workflow coordinator
/task workflow-coordinator "Add Priority field to Project screen"

# 3. The coordinator will:
#    - Pre-validate
#    - Generate scripts
#    - Execute changes
#    - Post-validate
#    - Generate rollback
```

### Bulk Field Addition
```bash
/task screen-designer "Add fields to Invoice: CustomerType (dropdown), Priority (int), Region (varchar), Notes (text)"
```

### Configuration Audit
```bash
/task validator "Perform complete audit of all custom fields and UDIC entities"
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