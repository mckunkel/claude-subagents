# Claude Sub-Agents

A collection of specialized sub-agents for Claude Code that automate complex workflows across different domains.

## Overview

This repository contains two main sub-agent ecosystems:

- **Hiring Sub-Agents**: Automated recruitment and resume optimization workflow
- **Fullstack Sub-Agents**: Complete software development lifecycle automation

## Structure

```
claude-subagents/
├── hiring-subagents/           # Recruitment workflow automation
│   ├── resume-analyzer/        # PDF resume analysis
│   ├── job-requirements-extractor/  # Job posting analysis
│   ├── skills-matcher/         # Skills gap analysis
│   ├── resume-optimizer/       # Resume content enhancement
│   ├── document-formatter/     # Multi-format document generation
│   └── hiring-workflow-orchestrator/  # End-to-end coordination
└── fullstack-subagents/        # Development workflow automation
    ├── product-manager/        # Requirements and feature analysis
    ├── technical-architect/    # System design and architecture
    ├── frontend-developer/     # UI/UX implementation
    ├── backend-developer/      # API and server-side development
    ├── devops-engineer/        # Infrastructure and deployment
    ├── qa-engineer/           # Testing and quality assurance
    ├── development-coordinator/ # Project coordination
    └── site-reliability-engineer/  # Monitoring and reliability
```

## Hiring Sub-Agents

Streamline the entire job application process from resume analysis to final document generation.

### Key Features

- **Automated Resume Analysis**: Extract and structure information from PDF resumes
- **Job Requirements Extraction**: Parse job postings and identify key requirements
- **Skills Matching**: Comprehensive analysis of candidate-job fit with gap identification
- **Resume Optimization**: AI-powered content enhancement with ATS optimization
- **Multi-Format Generation**: Professional documents in PDF, Word, and HTML formats
- **Workflow Orchestration**: End-to-end process management and coordination

### Workflow

1. **Analysis Phase**: Resume and job posting analysis
2. **Matching Phase**: Skills gap identification and recommendations
3. **Optimization Phase**: Content enhancement and keyword integration
4. **Generation Phase**: Professional document creation
5. **Coordination Phase**: Quality assurance and final delivery

## Fullstack Sub-Agents

Complete software development lifecycle automation with specialized agents for each role.

### Key Features

- **Product Management**: Requirements analysis, feature planning, and stakeholder coordination
- **Technical Architecture**: System design, technology selection, and scalability planning
- **Frontend Development**: UI/UX implementation with modern frameworks
- **Backend Development**: API design, database architecture, and server-side logic
- **DevOps Engineering**: CI/CD pipelines, infrastructure as code, and deployment automation
- **Quality Assurance**: Automated testing, performance validation, and quality metrics
- **Site Reliability**: Monitoring, alerting, and system reliability engineering

### Workflow Phases

1. **Phase 1**: Product analysis and requirements gathering
2. **Phase 2**: Technical architecture and system design
3. **Phase 3**: Implementation coordination across frontend and backend
4. **Phase 4**: Integration testing and quality assurance
5. **Phase 5**: Deployment and monitoring setup

## Getting Started

### Prerequisites

- Claude Code CLI
- Access to Claude API
- Relevant MCP tools for each sub-agent (see individual `tools.md` files)

### Installation

1. Clone this repository:
```bash
git clone https://github.com/mckunkel/claude-subagents.git
cd claude-subagents
```

2. Configure Claude Code with the desired sub-agents
3. Install required MCP tools based on the specific workflow you want to use

### Usage

Each sub-agent directory contains:
- `agent.md`: Detailed agent specification and capabilities
- `tools.md`: Required MCP tools and installation instructions
- Workflow-specific documentation and examples

## Contributing

1. Fork the repository
2. Create a feature branch
3. Add or modify sub-agents following the established patterns
4. Test your changes thoroughly
5. Submit a pull request

## Documentation

- [CLAUDE.md](./CLAUDE.md): Project-specific instructions and guidelines
- Individual agent documentation in respective directories
- Workflow examples and test results in dedicated files

## License

This project is open source and available under the MIT License.

## Support

For issues, questions, or contributions, please use the GitHub issue tracker.