# Autosubmit Scan Error Catalogs

This repository contains error detection catalogs and templates for the [autosubmit-scan](https://github.com/DestinE-Climate-DT/autosubmit-scan) tool.

## Repository Structure

```
.
├── templates/
│   └── default_autosubmit.yaml    # Default template for as-scan <expid>
└── examples/
    ├── production_catalog.yaml    # Production example
    └── development_catalog.yaml   # Development example
```

## Quick Start

### Using the Default Template

The simplest way to scan an Autosubmit experiment:

```bash
# Automatically uses the template from this repository
as-scan a23i
```

This will:
1. Load `templates/default_autosubmit.yaml` from this repository
2. Render it with your experiment ID (`a23i`)
3. Run the scan with comprehensive error detection
4. Generate a report

### Environment Variables

Configure default behavior with environment variables:

```bash
# Override default template location
export AUTOSUBMIT_SCAN_DEFAULT_TEMPLATE=github://DestinE-Climate-DT:autosubmit-scan-error-catalogs@v1.0/templates/default_autosubmit.yaml

# Override target host (default: mn5)
export AUTOSUBMIT_HOST=mn5

# Override base experiment path
export AUTOSUBMIT_BASE_PATH=/gpfs/scratch/ehpc01/awi478153
```

### Using Custom Catalogs

Load catalogs from any location:

```bash
# From this repository
as-scan scan --catalog github://DestinE-Climate-DT:autosubmit-scan-error-catalogs@main/examples/production_catalog.yaml

# From local file
as-scan scan --catalog /path/to/my_catalog.yaml

# From SSH
as-scan scan --catalog ssh://mn5:/shared/catalogs/production.yaml

# From S3
as-scan scan --catalog s3://bucket/catalogs/production.yaml

# From HTTPS
as-scan scan --catalog https://example.com/catalogs/production.yaml
```

## Templates

### default_autosubmit.yaml

The default template (`templates/default_autosubmit.yaml`) is used when you run `as-scan <expid>`.

**Template Variables:**

- `{{ expid }}` - Experiment ID (e.g., `a23i`, `t001`)
- `{{ user }}` - Current username
- `{{ host }}` - Target host (from `AUTOSUBMIT_HOST` or `mn5`)
- `{{ base_path }}` - Base experiment path (from `AUTOSUBMIT_BASE_PATH` or default)
- `{{ timestamp }}` - Current timestamp in ISO format

**Example Usage:**

```yaml
files:
  - ssh://{{ host }}:{{ base_path }}/{{ expid }}/LOG_{{ expid }}/*.out
  - ssh://{{ host }}:{{ base_path }}/{{ expid }}/LOG_{{ expid }}/*.err
```

Renders to:
```yaml
files:
  - ssh://mn5:/gpfs/scratch/ehpc01/awi478153/a23i/LOG_a23i/*.out
  - ssh://mn5:/gpfs/scratch/ehpc01/awi478153/a23i/LOG_a23i/*.err
```

### Detected Errors

The default template detects:

#### Critical Errors (severity: critical)
- **SlurmOutOfMemory** - Job killed due to memory limit
- **SlurmTimeLimit** - Job exceeded walltime
- **SlurmNodeFailure** - Node failure or launch problem
- **MissingInputFile** - Required input file not found
- **FortranRuntimeError** - Model crash or segfault
- **MPIError** - MPI communication failure

#### High Severity (severity: high)
- **PythonTraceback** - Python exception with traceback
- **PythonImportError** - Missing Python module
- **NetCDFError** - NetCDF file operation failed

#### Medium Severity (severity: medium)
- **PythonKeyError** - Dictionary key access error
- **ErrorKeyword** - Generic error keyword

### Railway Pattern

Some errors trigger follow-up checks. For example, detecting a `PythonTraceback` will automatically check for:
- `PythonImportError` (if traceback contains "ImportError")
- `PythonKeyError` (if traceback contains "KeyError")

## Examples

### Production Catalog

See `examples/production_catalog.yaml` for a complete production example with:
- Specific file paths
- Team assignments in metadata
- Railway pattern for cascading checks

### Development Catalog

See `examples/development_catalog.yaml` for a minimal testing example with:
- Local file paths (`/tmp/test_logs/`)
- Simple pattern matching
- Basic error categories

## Creating Custom Catalogs

### 1. Start with a Template

Copy an example:

```bash
cp examples/development_catalog.yaml my_catalog.yaml
```

### 2. Define Error Patterns

Each error definition includes:

```yaml
ErrorName:
  id: ErrorName                    # Unique identifier
  pattern:
    type: literal|regex|callable   # Pattern type
    pattern: "error text"          # Pattern to match
    flags: ["IGNORECASE"]          # Optional regex flags
  files:                           # Where to search
    - /path/to/logs/*.log
    - ssh://host:/path/*.err
  meaning: "What this error means"
  suggestion: "How to fix it"
  context_lines: 10                # Lines before/after match
  next_errors:                     # Railway pattern (optional)
    - error_id: FollowUpError
      when:
        type: always
  metadata:                        # Custom metadata
    severity: critical|high|medium|low
    category: resource|model|data|code
```

### 3. Pattern Types

**Literal** - Exact string match:
```yaml
pattern:
  type: literal
  pattern: "ERROR"
```

**Regex** - Regular expression:
```yaml
pattern:
  type: regex
  pattern: 'ERROR|FATAL|CRITICAL'
  flags: ["IGNORECASE", "MULTILINE"]
```

**Callable** - Custom Python function:
```yaml
pattern:
  type: callable
  pattern: "my_module:my_matcher_function"
```

### 4. File URIs

Supported protocols:
- Local: `/path/to/file.log`
- SSH: `ssh://user@host:/path/*.log`
- S3: `s3://bucket/path/*.log`
- FTP: `ftp://user:pass@host/path/*.log`
- GitHub: `github://org:repo@ref/path/file`

Glob patterns supported:
- `*.log` - All .log files
- `**/*.err` - Recursive search for .err files
- `LOG_*/job_*.out` - Pattern matching directories

### 5. Validate Your Catalog

```bash
as-scan validate my_catalog.yaml
```

### 6. Test Your Catalog

```bash
as-scan scan --catalog my_catalog.yaml --output ./results --dryrun
```

## Versioning

This repository follows semantic versioning:

- **main** - Latest stable version
- **develop** - Development branch
- **v1.0**, **v1.1**, etc. - Tagged releases

### Using Specific Versions

```bash
# Use specific version
export AUTOSUBMIT_SCAN_DEFAULT_TEMPLATE=github://DestinE-Climate-DT:autosubmit-scan-error-catalogs@v1.0/templates/default_autosubmit.yaml

# Use development branch
export AUTOSUBMIT_SCAN_DEFAULT_TEMPLATE=github://DestinE-Climate-DT:autosubmit-scan-error-catalogs@develop/templates/default_autosubmit.yaml
```

## Contributing

### Adding New Templates

1. Create template in `templates/` directory
2. Use Jinja2 syntax for variables
3. Document available variables in comments
4. Test with: `as-scan scan --catalog templates/your_template.yaml`

### Adding Examples

1. Create complete, working catalog in `examples/`
2. Use realistic file paths and patterns
3. Add descriptive comments
4. Include railway patterns if applicable

### Submitting Changes

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test thoroughly
5. Submit a pull request

## License

[License information to be added]

## Contact

- Issues: https://github.com/DestinE-Climate-DT/autosubmit-scan-error-catalogs/issues
- Main Project: https://github.com/DestinE-Climate-DT/autosubmit-scan

## Related Documentation

- [autosubmit-scan Documentation](https://github.com/DestinE-Climate-DT/autosubmit-scan)
- [Error Catalog Schema](https://github.com/DestinE-Climate-DT/autosubmit-scan/blob/main/docs/catalog_schema.md)
- [Pattern Matching Guide](https://github.com/DestinE-Climate-DT/autosubmit-scan/blob/main/docs/pattern_matching.md)
- [Railway Pattern Guide](https://github.com/DestinE-Climate-DT/autosubmit-scan/blob/main/docs/railway_pattern.md)
