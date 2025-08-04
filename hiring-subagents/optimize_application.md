# Hiring Application Optimization Command

## Command Overview

This command orchestrates the complete hiring sub-agents workflow to optimize a job application from resume analysis through final application strategy with interview preparation.

## Required Arguments

The command accepts three arguments in this format:
```
/optimize_application [resume_path] [job_source] [additional_skills]
```

**Arguments:**
- `$1` (resume_path): Path to resume file (PDF, Word, or text)
- `$2` (job_source): Job posting URL, file path, or job description text  
- `$3` (additional_skills): Optional comma-separated skills not on resume

## Prerequisites

The system SHALL verify that all required MCP tools are installed and configured:
- PDF Reader MCP (for resume and job posting parsing)
- Document Operations MCP (for professional document generation)
- LinkedIn MCP (for job discovery and networking, if job_source is LinkedIn URL)
- File Management (built-in Claude Code functionality)

## Workflow Execution

### Phase 1: Input Analysis and Validation

The system SHALL validate all inputs before proceeding:

**Resume Validation:**
- Verify the resume file exists and is accessible at the provided path
- Confirm file format is supported (PDF, DOC, DOCX, or TXT)
- Validate file integrity and readability

**Job Source Processing:**
- IF job_source contains "linkedin.com": Use LinkedIn MCP to extract job posting data
- ELSE IF job_source is a file path: Read and validate the job description file
- ELSE: Treat job_source as direct job description text

**Skills Processing:**
- Parse additional_skills parameter into structured skill list
- Validate skill names and remove duplicates
- Prepare skills supplement for analysis integration

### Phase 2: Core Analysis Workflow

The system SHALL execute the sub-agents in sequence:

**Step 1: Resume Analysis**
```
Use the resume-analyzer sub-agent to:
- Parse the resume file at $1 (resume_path)
- Extract structured candidate data with skills, experience, education
- Generate candidate profile JSON with parsing confidence metrics
- Validate extraction quality and completeness
```

**Step 2: Job Requirements Extraction**
```
Use the job-extractor sub-agent to:
- Process job posting from $2 (job_source)
- Extract detailed requirements, qualifications, and company information
- Generate structured job requirements JSON
- Identify critical vs. preferred qualifications
```

**Step 3: Skills Matching and Gap Analysis**
```
Use the skills-matcher sub-agent to:
- Compare candidate profile against job requirements
- Integrate additional skills from $3 into analysis
- Calculate match scores and identify optimization opportunities
- Generate comprehensive recommendations for improvement
```

### Phase 3: Optimization and Document Generation

**Step 4: Content Optimization**
```
Use the resume-optimizer sub-agent to:
- Generate enhanced resume content based on skills matching analysis
- Integrate job-specific keywords and achievements
- Apply ATS optimization techniques
- Create compelling value propositions aligned with job requirements
```

**Step 5: Professional Document Creation**
```
Use the document-formatter sub-agent to:
- Generate professional resume documents in multiple formats (PDF, Word, HTML, Text)
- Apply ATS-compatible formatting and styling
- Ensure consistent branding and professional appearance
- Validate document quality and parsing accuracy
```

### Phase 4: Strategic Coordination and Delivery

**Step 6: Application Strategy Development**
```
Use the application-coordinator sub-agent to:
- Orchestrate complete application workflow
- Generate strategic application timeline and approach
- Create comprehensive interview preparation materials
- Develop networking and follow-up strategies
- Calculate success probability and competitive positioning
```

## Output Generation

The system SHALL generate a comprehensive application package:

**Optimized Documents:**
- Primary resume in PDF format (ATS-optimized)
- Editable Word version for customization
- HTML version for online applications
- Plain text version for ATS systems

**Strategic Materials:**
- Interview preparation guide with role-specific focus
- Skills development roadmap with learning priorities
- Networking strategy with target connections
- Application timeline with optimal submission timing

**Analysis Reports:**
- Skills match analysis with gap identification
- Market intelligence and salary insights
- Competitive positioning assessment
- Success probability metrics

## Quality Assurance Requirements

The system SHALL validate each phase before proceeding:

**Data Integrity Checks:**
- Verify JSON schema compliance between agents
- Validate data consistency across workflow stages
- Confirm no critical information loss during processing

**Quality Metrics Validation:**
- Resume parsing confidence > 95%
- Skills match score calculation accuracy
- ATS compatibility score > 90%
- Document formatting professional standards

**Error Handling:**
- Graceful degradation for partial failures
- Clear error messages with resolution guidance
- Fallback strategies for tool unavailability
- Progress preservation for workflow recovery

## Success Criteria

The workflow is considered successful when:
- All six sub-agents complete processing without critical errors
- Skills match score is calculated and improvement recommendations provided
- Professional documents are generated in all required formats
- Application strategy is developed with clear action items
- Total processing time remains under 60 minutes

## Output Structure

The system SHALL organize outputs in the following structure:
```
application-package-[timestamp]/
├── documents/
│   ├── resume-optimized.pdf
│   ├── resume-optimized.docx
│   ├── resume-optimized.html
│   └── resume-optimized.txt
├── analysis/
│   ├── skills-match-report.json
│   ├── market-intelligence.json
│   └── competitive-analysis.json
├── strategy/
│   ├── interview-preparation.md
│   ├── application-timeline.md
│   └── networking-strategy.md
└── summary/
    └── optimization-summary.md
```

## Usage Examples

**Basic Usage:**
```bash
/optimize_application ~/Documents/resume.pdf "https://linkedin.com/jobs/view/12345" "GraphQL,Docker,Kubernetes"
```

**File-based Job Description:**
```bash
/optimize_application ./my-resume.pdf ./job-posting.txt "Machine Learning,Python,AWS"
```

**Direct Text Job Description:**
```bash
/optimize_application resume.pdf "Senior Software Engineer at TechCorp - React, Node.js, 5+ years experience required" "TypeScript,PostgreSQL"
```

## Error Recovery

IF any sub-agent fails:
1. Document the failure point and error details
2. Attempt to continue with remaining agents using available data
3. Generate partial results with quality warnings
4. Provide clear guidance on manual completion steps
5. Preserve all intermediate results for troubleshooting

## Performance Optimization

The system MAY implement performance enhancements:
- Parallel processing of independent analysis tasks
- Caching of frequently analyzed job postings
- Incremental updates for resume modifications
- Predictive pre-processing of common job types

## Integration Requirements

The command SHALL integrate seamlessly with:
- Claude Code sub-agent system
- MCP tool ecosystem
- Local file system for document storage
- Web services for LinkedIn integration (when applicable)

Remember: This command provides end-to-end automation of the job application optimization process, transforming a basic resume and job posting into a comprehensive, strategically optimized application package with professional documentation and interview preparation materials.