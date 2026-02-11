# CodeJanitor 2.0 - Requirements Document

## 1. Executive Summary

**Project Name:** CodeJanitor 2.0  
**Version:** 2.0.0  
**Status:** Production Ready  
**Purpose:** Enterprise-grade autonomous security remediation system that detects, verifies, and automatically fixes security vulnerabilities in code repositories.

### Vision Statement
CodeJanitor automates the entire security vulnerability lifecycle - from detection through verification to remediation - using a multi-agent AI system with human-in-the-loop control.

### Key Innovation
**Test-Driven Repair Pattern**: Vulnerabilities are verified by generating working exploits (Red Team), then patches are validated by ensuring those exploits fail (Blue Team).

---

## 2. System Overview

### 2.1 Core Workflow
```
Repository Input → Security Audit → Red Team Verification → Blue Team Patching → Pull Request Creation
```

### 2.2 Architecture Pattern
Multi-agent system with:
- **Auditor Agent**: Scans code for vulnerabilities
- **Red Team Agent**: Generates and executes exploits to verify vulnerabilities
- **Blue Team Agent**: Creates and validates security patches
- **Orchestrator**: Coordinates the workflow
- **Knowledge Graph**: Provides dependency context

---

## 3. Functional Requirements

### 3.1 Security Scanning (FR-SCAN)

**FR-SCAN-001**: Repository Cloning
- System SHALL clone GitHub repositories via HTTPS
- System SHALL support authentication via GitHub tokens
- System SHALL handle both public and private repositories

**FR-SCAN-002**: Code Parsing
- System SHALL parse Python source files using Tree-sitter
- System SHALL extract function definitions with metadata (name, line numbers, docstrings)
- System SHALL handle syntax errors gracefully

**FR-SCAN-003**: Vulnerability Detection
- System SHALL analyze code for security vulnerabilities using LLM
- System SHALL detect at minimum:
  - SQL Injection
  - Command Injection
  - Path Traversal
  - Insecure Deserialization
  - Weak Cryptography
  - Hardcoded Credentials
  - Insecure Randomness
- System SHALL assign risk scores (0-10) to each finding
- System SHALL provide detailed descriptions of vulnerabilities

**FR-SCAN-004**: Knowledge Graph Construction
- System SHALL build dependency graph of entire codebase
- System SHALL resolve import statements (absolute and relative)
- System SHALL provide context-aware code retrieval
- System SHALL support multi-level dependency traversal

### 3.2 Red Team Verification (FR-RED)

**FR-RED-001**: Kill Chain Methodology
- System SHALL follow RECON → PLAN → EXPLOIT methodology
- System SHALL document reasoning at each phase
- System SHALL generate proof-of-concept exploit code

**FR-RED-002**: Exploit Execution
- System SHALL execute exploits in isolated Docker containers
- System SHALL inject target code and dependencies into sandbox
- System SHALL capture stdout, stderr, and exit codes
- System SHALL enforce timeout limits (default: 15 seconds)

**FR-RED-003**: Verification Logic
- System SHALL determine exploit success based on:
  - Explicit "EXPLOIT_SUCCESS" markers
  - Exit codes
  - Output patterns
- System SHALL mark unverified issues as false positives

**FR-RED-004**: Iterative Reasoning
- System SHALL retry failed exploits up to 3 times
- System SHALL learn from previous failed attempts
- System SHALL modify strategy based on failure history

### 3.3 Blue Team Remediation (FR-BLUE)

**FR-BLUE-001**: Patch Generation
- System SHALL generate secure code replacements using LLM
- System SHALL maintain function signatures and behavior
- System SHALL use secure coding patterns:
  - Parameterized queries for SQL
  - Subprocess with list arguments for commands
  - Path validation for file operations
  - Strong cryptography (bcrypt, argon2)
  - Environment variables for secrets

**FR-BLUE-002**: Test-Driven Repair
- System SHALL verify exploit works on vulnerable code (baseline)
- System SHALL apply patch
- System SHALL verify exploit fails on patched code
- System SHALL retry patch generation up to 3 times if verification fails

**FR-BLUE-003**: Context-Aware Patching
- System SHALL use knowledge graph for dependency context
- System SHALL preserve all imports
- System SHALL maintain code structure

### 3.4 Git Operations (FR-GIT)

**FR-GIT-001**: Fork-Based Workflow
- System SHALL fork target repository to authenticated user's account
- System SHALL reuse existing forks if available
- System SHALL clone fork (not original repository)

**FR-GIT-002**: Branch Management
- System SHALL create fix branches named `codejanitor/fix-{issue_id}`
- System SHALL commit patched code with descriptive messages
- System SHALL push branches to fork

**FR-GIT-003**: Pull Request Creation
- System SHALL create PRs from fork to original repository
- System SHALL include:
  - Descriptive title with vulnerability type
  - Verification status
  - Link to issue number
  - Automated testing results
- System SHALL handle existing PRs gracefully

### 3.5 User Interface (FR-UI)

**FR-UI-001**: Interactive CLI
- System SHALL provide Rich-based terminal interface
- System SHALL display:
  - Vulnerability risk reports (tables)
  - Real-time progress indicators
  - Phase-specific panels (Red Team, Blue Team, Git)
- System SHALL support user actions:
  - Fix: Execute full workflow
  - Skip: Move to next issue
  - Exit: Stop execution

**FR-UI-002**: Web Dashboard
- System SHALL provide Next.js-based web interface
- System SHALL support:
  - Repository scanning
  - Vulnerability list with selection
  - Live remediation console with streaming logs
  - PR result display
- System SHALL use real-time API streaming for progress updates

**FR-UI-003**: Prioritization Strategies
- System SHALL offer two modes:
  - System Mode: AI sorts by risk score
  - Manual Mode: User selects specific issues
- System SHALL display issues sorted by severity

### 3.6 API Layer (FR-API)

**FR-API-001**: REST Endpoints
- System SHALL provide `/api/scan` endpoint for repository scanning
- System SHALL provide `/api/fix` endpoint for vulnerability remediation
- System SHALL support CORS for frontend integration

**FR-API-002**: Streaming Responses
- System SHALL stream logs via NDJSON format
- System SHALL emit phase-specific messages (red_team, blue_team, pr)
- System SHALL provide final success/failure status with PR URL

---

## 4. Non-Functional Requirements

### 4.1 Performance (NFR-PERF)

**NFR-PERF-001**: Scanning Speed
- System SHALL scan 5-10 Python files per second
- System SHALL complete knowledge graph construction in <30 seconds for typical repositories

**NFR-PERF-002**: Exploit Execution
- System SHALL execute exploits within 15 seconds (configurable)
- System SHALL support parallel container execution

**NFR-PERF-003**: Patch Generation
- System SHALL generate patches within 20 seconds per attempt
- System SHALL complete full fix workflow in <2 minutes per vulnerability

### 4.2 Security (NFR-SEC)

**NFR-SEC-001**: Isolation
- System SHALL execute all exploit code in Docker containers
- System SHALL disable network access in containers by default
- System SHALL enforce memory limits (512MB default)
- System SHALL automatically cleanup containers after execution

**NFR-SEC-002**: Data Protection
- System SHALL store GitHub tokens in environment variables only
- System SHALL not log sensitive data
- System SHALL delete temporary directories after use
- System SHALL not persist exploit code

**NFR-SEC-003**: Access Control
- System SHALL require GitHub token with `repo` scope for PR creation
- System SHALL use fork-based workflow to avoid direct repository access

### 4.3 Reliability (NFR-REL)

**NFR-REL-001**: Error Handling
- System SHALL handle LLM failures with retry logic (3 attempts)
- System SHALL gracefully handle Docker unavailability
- System SHALL continue execution if individual file scans fail
- System SHALL provide detailed error messages

**NFR-REL-002**: Resilience
- System SHALL support LLM fallback (Groq → OpenAI)
- System SHALL handle network timeouts
- System SHALL recover from partial failures

**NFR-REL-003**: Testing
- System SHALL maintain 100% test pass rate (81/81 tests)
- System SHALL cover all core components with unit tests
- System SHALL include integration tests for workflows

### 4.4 Scalability (NFR-SCALE)

**NFR-SCALE-001**: Repository Size
- System SHALL support repositories up to 10,000 files
- System SHALL handle dependency graphs with 1,000+ nodes

**NFR-SCALE-002**: Concurrent Operations
- System SHALL support multiple scan sessions
- System SHALL isolate session data in temporary directories

### 4.5 Usability (NFR-USE)

**NFR-USE-001**: User Experience
- System SHALL provide clear progress indicators
- System SHALL display actionable error messages
- System SHALL support multiple input formats for repository URLs

**NFR-USE-002**: Documentation
- System SHALL include comprehensive README
- System SHALL provide quick start guide
- System SHALL document all configuration options

### 4.6 Maintainability (NFR-MAINT)

**NFR-MAINT-001**: Code Quality
- System SHALL follow PEP 8 style guidelines
- System SHALL use type hints for function signatures
- System SHALL include docstrings for all public functions

**NFR-MAINT-002**: Configuration
- System SHALL use Pydantic for settings management
- System SHALL support .env file configuration
- System SHALL provide sensible defaults

---

## 5. Technical Requirements

### 5.1 Technology Stack (TR-TECH)

**TR-TECH-001**: Core Technologies
- Python 3.11+
- LangChain for LLM orchestration
- Groq API for fast LLM inference
- Docker for sandboxed execution
- Tree-sitter for code parsing
- NetworkX for dependency graphs

**TR-TECH-002**: Frontend Technologies
- Next.js 14
- React 18
- TypeScript 5
- Tailwind CSS
- Framer Motion for animations

**TR-TECH-003**: Infrastructure
- FastAPI for REST API
- Uvicorn for ASGI server
- GitPython for Git operations
- PyGithub for GitHub API

### 5.2 Dependencies (TR-DEP)

**TR-DEP-001**: Required Services
- Docker daemon (running)
- GitHub account with token
- Groq API key (or OpenAI as fallback)

**TR-DEP-002**: Python Packages
- See requirements.txt for complete list
- Key dependencies:
  - langgraph>=0.2.0
  - langchain>=0.3.0
  - docker>=7.0.0
  - tree-sitter>=0.21.0
  - networkx>=3.0
  - fastapi>=0.109.0

### 5.3 Environment (TR-ENV)

**TR-ENV-001**: Operating Systems
- System SHALL support Windows, Linux, macOS
- System SHALL adapt shell commands to platform

**TR-ENV-002**: Resource Requirements
- Minimum 4GB RAM
- Docker with 512MB per container
- 1GB disk space for Docker images

---

## 6. Data Requirements

### 6.1 Input Data (DR-INPUT)

**DR-INPUT-001**: Repository Data
- GitHub repository URL (various formats supported)
- GitHub token (optional for public repos)
- File patterns for scanning (default: **/*.py)

**DR-INPUT-002**: Configuration Data
- LLM model selection
- Docker image name
- Timeout values
- Memory limits

### 6.2 Output Data (DR-OUTPUT)

**DR-OUTPUT-001**: Scan Results
- Vulnerability list with:
  - ID, type, severity, file, line, risk score, description
- Summary statistics:
  - Total vulnerabilities
  - Severity breakdown
  - Files scanned

**DR-OUTPUT-002**: Remediation Results
- PR URL
- Verification status
- Attempt count
- Error messages (if failed)

**DR-OUTPUT-003**: Logs
- Phase-specific logs (recon, plan, exploit, defense, patch, verify)
- Execution timestamps
- Error traces

### 6.3 Persistent Data (DR-PERSIST)

**DR-PERSIST-001**: Session Data
- Temporary directories for cloned repositories
- Knowledge graph in memory
- Active session metadata

**DR-PERSIST-002**: Configuration
- .env file for secrets
- Settings in Pydantic models

---

## 7. Integration Requirements

### 7.1 External Services (IR-EXT)

**IR-EXT-001**: GitHub Integration
- System SHALL integrate with GitHub REST API v3
- System SHALL support repository operations (clone, fork, PR)
- System SHALL handle rate limiting

**IR-EXT-002**: LLM Integration
- System SHALL integrate with Groq API
- System SHALL support OpenAI as fallback
- System SHALL handle API errors gracefully

**IR-EXT-003**: Docker Integration
- System SHALL use Docker SDK for Python
- System SHALL manage container lifecycle
- System SHALL handle image pulling

### 7.2 Internal Integration (IR-INT)

**IR-INT-001**: Component Communication
- Orchestrator SHALL coordinate all agents
- Agents SHALL communicate via return values
- Knowledge graph SHALL be shared across agents

**IR-INT-002**: Frontend-Backend Communication
- Frontend SHALL call REST API endpoints
- Backend SHALL stream responses via NDJSON
- CORS SHALL be configured for localhost

---

## 8. Compliance Requirements

### 8.1 Security Standards (CR-SEC)

**CR-SEC-001**: OWASP Compliance
- System SHALL detect OWASP Top 10 vulnerabilities
- System SHALL follow secure coding practices in patches

**CR-SEC-002**: Data Privacy
- System SHALL not store repository code permanently
- System SHALL not transmit code to third parties (except LLM APIs)

### 8.2 Licensing (CR-LIC)

**CR-LIC-001**: Open Source
- System SHALL use MIT license
- System SHALL comply with dependency licenses

---

## 9. Constraints

### 9.1 Technical Constraints (C-TECH)

**C-TECH-001**: Language Support
- System SHALL support Python only (v2.0)
- Future versions MAY support JavaScript, Java, Go

**C-TECH-002**: LLM Limitations
- System SHALL depend on LLM availability
- System SHALL handle LLM rate limits
- Patch quality depends on LLM capabilities

### 9.2 Operational Constraints (C-OPS)

**C-OPS-001**: Docker Requirement
- System SHALL require Docker daemon running
- System SHALL not function without Docker

**C-OPS-002**: Network Requirement
- System SHALL require internet for:
  - Repository cloning
  - LLM API calls
  - GitHub API calls

---

## 10. Success Criteria

### 10.1 Functional Success (SC-FUNC)

**SC-FUNC-001**: Detection Accuracy
- False positive rate <5% (with Red Team verification)
- Detection coverage for OWASP Top 10

**SC-FUNC-002**: Patch Success Rate
- 85% success rate on first attempt
- 95% success rate within 3 attempts

**SC-FUNC-003**: End-to-End Workflow
- Complete workflow (scan → verify → patch → PR) in <5 minutes per vulnerability

### 10.2 Quality Success (SC-QUAL)

**SC-QUAL-001**: Test Coverage
- 100% test pass rate
- All core components covered

**SC-QUAL-002**: Code Quality
- No critical security issues in codebase
- PEP 8 compliance

### 10.3 User Success (SC-USER)

**SC-USER-001**: Usability
- Users can complete workflow without documentation
- Clear error messages for all failure modes

**SC-USER-002**: Adoption
- Successful deployment in production environments
- Positive user feedback

---

## 11. Future Enhancements

### 11.1 Planned Features (FE-PLAN)

**FE-PLAN-001**: Multi-Language Support
- JavaScript/TypeScript
- Java
- Go

**FE-PLAN-002**: CI/CD Integration
- GitHub Actions workflow
- GitLab CI integration
- Jenkins plugin

**FE-PLAN-003**: Advanced Features
- Historical vulnerability tracking
- Metrics dashboard
- Slack/Discord notifications
- Webhook integration

### 11.2 Research Areas (FE-RESEARCH)

**FE-RESEARCH-001**: Advanced Analysis
- Call graph analysis
- Data flow tracking
- Taint analysis

**FE-RESEARCH-002**: Performance
- Parallel Red Team verification
- Exploit caching
- Incremental scanning

---

## 12. Glossary

| Term | Definition |
|------|------------|
| Red Team | Offensive security agent that generates exploits |
| Blue Team | Defensive security agent that creates patches |
| Kill Chain | RECON → PLAN → EXPLOIT methodology |
| Test-Driven Repair | Verify exploit works, apply patch, verify exploit fails |
| Knowledge Graph | Dependency graph of codebase |
| Fork-Based Workflow | Create PR from fork to original repository |
| NDJSON | Newline-delimited JSON for streaming |

---

## 13. Approval

| Role | Name | Signature | Date |
|------|------|-----------|------|
| Product Owner | | | |
| Technical Lead | | | |
| Security Lead | | | |

---

**Document Version:** 1.0  
**Last Updated:** 2024  
**Status:** Final
