# Vantagepoint Marketplace

A Claude Code plugin marketplace providing professional tools for Vantagepoint development and UI configuration.

## Quick Start

### Installation

```bash
# Add the marketplace to Claude Code
/plugin marketplace add owner/repo

# Install the vantagepoint-tools plugin
/plugin install vantagepoint-tools@vantagepoint-marketplace
```

### Available Plugins

#### Vantagepoint Tools
A comprehensive toolkit containing four specialized agents for Vantagepoint UI configuration:

- **screen-designer**: SQL script generation for UI configuration
- **rollback**: Safe rollback script generation
- **validator**: Configuration validation and troubleshooting
- **workflow-coordinator**: Multi-step workflow orchestration

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
Streamlines Vantagepoint UI configuration with intelligent agents that prevent common errors and automate complex workflows.

### Key Features
- **Automated GUID Generation**: Properly formatted GUIDs without hyphens
- **TabID Validation**: Prevents mismatches across configuration tables
- **Dependency Management**: Checks dependencies before modifications
- **Rollback Safety**: Automatic rollback script generation
- **Workflow Automation**: Orchestrates multi-step operations

### Agent Capabilities

#### Screen Designer
- Custom field creation
- UDIC entity management
- Screen layout modifications
- Divider and separator configuration
- SEField entry management

#### Rollback Generator
- Safe field removal with dependency checking
- Configuration restoration
- Backup table creation
- Audit trail preservation

#### Validator
- Configuration completeness verification
- Missing component identification
- Permission validation
- Performance impact assessment

#### Workflow Coordinator
- Multi-agent orchestration
- Sequential operation management
- Error recovery
- State management across operations

## Usage Examples

### Add a Custom Field
```bash
/task screen-designer "Add CustomerType field to Invoice screen"
```

### Validate Configuration
```bash
/task validator "Check Invoice screen configuration"
```

### Complex Workflow
```bash
/task workflow-coordinator "Add multiple fields with validation and rollback"
```

### Generate Rollback
```bash
/task rollback "Create rollback script for today's changes"
```

## Development

### Adding New Plugins

1. Create a new plugin directory
2. Add `.claude-plugin/plugin.json`
3. Add your agents, commands, or hooks
4. Update `marketplace.json` to include your plugin
5. Test locally before publishing

### Testing Locally

```bash
# Install from local marketplace
/plugin install vantagepoint-tools@./.claude-plugin/marketplace.json
```

## Best Practices

### For Users
1. Always validate before making changes
2. Generate rollback scripts after modifications
3. Use workflow coordinator for complex operations
4. Test in development environment first

### For Plugin Developers
1. Follow the established agent patterns
2. Include comprehensive documentation
3. Provide clear error messages
4. Include example usage in agent definitions

## Requirements

### Database Access
Vantagepoint tools require access to:
- MS SQL Server database
- FW_CustomColumnsData table
- SEFields table
- UDICEntity tables
- FW_TabConfig table

### MCP Tools
Ensure these MCP tools are configured:
- mcp__MSSQL__describe_table
- mcp__MSSQL__read_data
- mcp__MSSQL__insert_data
- mcp__MSSQL__update_data

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

# Make changes and test locally
/plugin install vantagepoint-tools@./.claude-plugin/marketplace.json

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
