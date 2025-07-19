# vAin - Vulnerability AI Navigator

## Overview
vAin is an AI-powered vulnerability analysis system that automates the ingestion and processing of vulnerability reports and codebases. It uses advanced LLM capabilities to extract findings, generate detection templates using the Primer methodology, and manage vulnerability data for security research.

## Current Features
- **Report Ingestion**: Support for vulnerability reports from multiple sources (.md, .pdf, URLs)
- **GitHub Integration**: 
  - Ingest GitHub Issues as vulnerability reports with filtering options
  - Ingest vulnerability reports from GitHub repositories
  - API key support to prevent rate limiting
- **Codebase Management**: Ingest and manage codebases from local paths or GitHub URLs
- **Primer Methodology**: Advanced template generation and refinement using AI-driven primers
- **Template Management**: Create, extract, and analyze detection templates
- **Data Processing Pipeline**: Parse reports, extract findings, and vectorize for semantic search
- **Report-Codebase Linking**: Link vulnerability reports to their corresponding codebases
- **LLM Integration**: Support for multiple backends (Ollama local and OpenRouter cloud)
- **Database Operations**: Comprehensive data management and cleanup utilities

## Getting Started

### Setup
```bash
# Setup virtual environment
uv venv venv
source venv/bin/activate

# Install dependencies
uv pip install -r requirements.txt

# Configure LLM provider (required for AI features)
uv run vain.py --configure-llm
```

### Core Workflows

#### Report Ingestion
```bash
# Ingest a local vulnerability report
uv run vain.py --ingest-report /path/to/report.md

# Ingest from URL
uv run vain.py --ingest-report https://example.com/vuln-report.pdf

# Ingest GitHub Issues as reports
uv run vain.py --ingest-github-issues owner/repo --issue-labels security,bug

# Ingest reports from GitHub repository
uv run vain.py --ingest-github-repo --github-owner owner --github-repo repo --github-token YOUR_TOKEN
```

#### Codebase Management
```bash
# Ingest a codebase
uv run vain.py --ingest-codebase https://github.com/owner/repo
uv run vain.py --ingest-codebase /path/to/local/code

# List available codebases
uv run vain.py --list-codebases

# Remove a codebase (interactive)
uv run vain.py --remove-codebase
```

#### Report-Codebase Linking
```bash
# Interactive linking
uv run vain.py --link-reports

# Link specific report to codebase
uv run vain.py --link-report REPORT_ID CODEBASE_ID

# List unlinked reports
uv run vain.py --list-unlinked-reports
```

#### Primer and Template Management
```bash
# List vulnerability types from findings
uv run vain.py --list-vuln-types

# Create a primer for specific vulnerability type
uv run vain.py --seed-primer "reentrancy"

# Batch create primers for all vulnerability types
uv run vain.py --batch-create-primers --primer-limit 10

# Generate detection templates using primers
uv run vain.py --generate

# Extract template from specific primer
uv run vain.py --extract-template primers/reentrancy_primer_v1.md

# List existing primers and templates
uv run vain.py --list-primers
uv run vain.py --list-templates

# Analyze primers for similarity
uv run vain.py --analyze-primers --similarity-threshold 0.8
```

#### Data Processing
```bash
# Run full data processing pipeline
uv run vain.py --all

# Individual pipeline steps
uv run vain.py --parse          # Parse reports
uv run vain.py --map            # Map findings to code
uv run vain.py --vectorize      # Vectorize findings

# Extract findings from reports
uv run vain.py --extract-findings --limit 50
```

#### Database Management
```bash
# Clear specific data types
uv run vain.py --clear-findings
uv run vain.py --clear-reports
uv run vain.py --clear-all

# Skip confirmation prompts
uv run vain.py --clear-all --skip-confirmation
```

## Core Components
- `vain.py` - Main entry point and CLI interface
- `unified_pipeline.py` - Core data processing pipeline
- `primer.py` - Primer methodology implementation
- `llm_provider.py` - LLM backend integration (Ollama/OpenRouter)
- `github_api.py` - GitHub API integration with rate limit handling
- `link_reports_codebases.py` - Report-codebase linking functionality
- `protocol_detector.py` - Protocol type detection utilities
- `dynamic_ingestion.py` - Advanced report parsing and finding extraction

## Database Schema
The project uses SQLite with the following main tables:
- `reports` - Vulnerability report metadata
- `findings` - Individual vulnerability findings extracted from reports
- `codebases` - Ingested codebase information
- `report_codebase_links` - Associations between reports and codebases

## Configuration

### LLM Providers
vAin supports two LLM backends:
- **Ollama** (local): For privacy-focused local inference
- **OpenRouter** (cloud): For access to various commercial models

Configure your preferred provider:
```bash
uv run vain.py --configure-llm
```

### GitHub API
For GitHub repository ingestion and to avoid rate limits:
```bash
# Set GitHub token for private repos or rate limit avoidance
uv run vain.py --ingest-github-repo --github-token YOUR_GITHUB_TOKEN
```

## Architecture
The project follows a modular architecture with clear separation between:
- Data ingestion and processing
- LLM-powered analysis and template generation
- Database management and querying
- CLI interface and user interaction

For detailed architecture information, see [ai_vuln_bot_plan.md](ai_vuln_bot_plan.md).

## 📜 License
MIT License © 2025 pxng0lin/ThΞ CxgΞ
