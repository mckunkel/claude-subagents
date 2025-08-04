# MCP Tools Analysis for Hiring Sub-Agents

## Tool Priority Rating System
- **10**: Absolutely essential for core functionality
- **8-9**: Highly important for enhanced features
- **6-7**: Useful for improved user experience
- **4-5**: Nice to have for additional capabilities
- **1-3**: Optional or experimental features

## Essential Tools (Priority 8-10)

### 1. PDF Reader MCP - Priority: 10
**Purpose**: Resume and job description parsing from PDF files
**GitHub**: https://github.com/sylphxltd/pdf-reader-mcp
**Provider**: SylphLab
**Classification**: Professional/Community

#### Capabilities
- Extract text content from PDF files (local and URLs)
- Extract metadata (title, author, creation date)
- Get page counts
- Support for specific page extraction
- Structured JSON output format

#### Installation Options
**Option A: NPM (Recommended)**
```bash
npm install @sylphlab/pdf-reader-mcp
```

**Claude Desktop Configuration**:
```json
{
  "mcpServers": {
    "pdf-reader-mcp": {
      "command": "npx",
      "args": ["@sylphlab/pdf-reader-mcp"],
      "name": "PDF Reader"
    }
  }
}
```

**Option B: Docker**
```bash
docker pull sylphlab/pdf-reader-mcp:latest
```

#### Integration Notes
- Single `read_pdf` tool handles multiple extraction needs
- Returns predictable JSON format for easy agent parsing
- Supports both local files and URL-based PDFs
- Essential for Resume Analyzer and Job Requirements Extractor agents

---

### 2. Document Operations MCP - Priority: 9
**Purpose**: Professional document generation (Word, PDF, Excel)
**GitHub**: https://github.com/alejandroballesterosc/document-edit-mcp
**Provider**: Alejandro Ballesteros
**Classification**: Community

#### Capabilities
- Create and edit Microsoft Word documents
- Generate PDF files with formatting
- Excel spreadsheet operations
- Document conversion between formats
- Template-based document generation

#### Installation
```bash
# Using FastMCP framework
pip install fastmcp
git clone https://github.com/alejandroballesterosc/document-edit-mcp
cd document-edit-mcp
pip install -r requirements.txt
```

**Claude Desktop Configuration**:
```json
{
  "mcpServers": {
    "document-operations": {
      "command": "python",
      "args": ["/path/to/document-edit-mcp/server.py"]
    }
  }
}
```

#### Integration Notes
- Critical for Document Formatter agent
- Enables professional resume output in multiple formats
- Supports template-based generation for consistent formatting
- Python-based with FastMCP integration

---

### 3. Career Site Job Listing API MCP - Priority: 9
**Purpose**: Real job postings from 105k+ company career sites
**GitHub**: https://mcpservers.org/servers/fantastic-jobs/career-site-job-listing-api
**Provider**: Fantastic.jobs B.V. (via Apify)
**Classification**: Professional/Commercial

#### Capabilities
- Direct job postings from 35+ ATS platforms (Workday, Greenhouse, Ashby, etc.)
- AI-enriched with LinkedIn company data
- Up to 60 fields per job posting including:
  - Job title, description, requirements
  - Company information and size
  - Location and remote options
  - Salary ranges (when available)
  - Application URLs and contact info
- Real-time, high-quality job data
- Search by keywords, location, company

#### Installation
**Via Apify MCP Server**
```bash
npm install -g mcp-remote
```

**Claude Desktop Configuration**:
```json
{
  "mcpServers": {
    "apify-jobs": {
      "command": "npx",
      "args": [
        "mcp-remote",
        "https://mcp.apify.com/sse?actors=fantastic-jobs/career-site-job-listing-api",
        "--header",
        "Authorization: Bearer <YOUR_APIFY_API_TOKEN>"
      ]
    }
  }
}
```

#### Pricing
- **Free Tier**: Available for testing
- **Paid**: From $6.00 per 1,000 jobs
- **API Token**: Required (get from Apify Console)

#### Integration Notes
- Legitimate API (no TOS violations like LinkedIn scraping)
- Perfect for Job Requirements Extractor with structured data
- Enables automated job discovery for Application Coordinator
- Provides comprehensive job market data for Skills Matcher agent
- Much more reliable than unofficial LinkedIn scraping tools

---

### 4. LinkedIn MCP - Priority: 6 (Deprecated)
**Purpose**: ~~Job posting discovery~~ - Professional profile analysis only
**GitHub**: https://github.com/adhikasp/mcp-linkedin
**Provider**: adhikasp
**Classification**: Community
**Status**: ⚠️ NOT RECOMMENDED for job search

#### Security Considerations
- Uses unofficial LinkedIn API (violates LinkedIn TOS)
- Requires personal LinkedIn credentials
- High risk of account suspension
- May be subject to rate limiting

#### Recommendation
**Use Career Site Job Listing API instead** for job discovery. Only consider LinkedIn MCP for non-job related professional profile analysis if absolutely necessary.

---

## Supplementary Tools (Priority 6-7)

### 4. PDF Manipulation MCP - Priority: 7
**Purpose**: Advanced PDF operations (merging, splitting, page extraction)
**GitHub**: https://github.com/hanweg/mcp-pdf-tools
**Provider**: hanweg
**Status**: Work in Progress (Windows focus)

#### Capabilities
- Merge multiple PDF files
- Extract specific pages
- Search PDFs with regex patterns
- Find related PDFs based on content
- Order-specific PDF merging

#### Installation
```bash
git clone https://github.com/hanweg/mcp-pdf-tools
cd mcp-pdf-tools
uv venv
source .venv/bin/activate  # or .venv\Scripts\activate on Windows
uv pip install -e .
```

**Configuration**:
```json
{
  "mcpServers": {
    "pdf-tools": {
      "command": "uv",
      "args": ["--directory", "/path/to/mcp-pdf-tools", "run", "pdf-tools"]
    }
  }
}
```

#### Integration Notes
- Useful for combining multiple resume versions
- Can extract specific sections from complex documents
- Currently Windows-focused (may need adaptation for macOS)

---

### 5. File Management MCP - Priority: 6
**Purpose**: Local file operations and storage management
**Built-in**: Claude Code native capabilities
**Classification**: Core

#### Capabilities
- Read and write local files
- Directory listing and navigation
- File copying and moving
- Basic file metadata operations

#### Integration Notes
- Uses built-in Claude Code tools (Read, Write, LS)
- No additional installation required
- Handles temporary file storage between agents
- Manages input/output file workflows

---

## Installation Priority and Recommendations

### Phase 1 Implementation (Required)
1. **PDF Reader MCP** - Start with NPM installation for reliability
2. **File Management** - Already available, no setup needed

### Phase 2 Implementation (Enhanced Features)
3. **Document Operations MCP** - For professional output generation
4. **Career Site Job Listing API MCP** - For legitimate job discovery automation

### Phase 3 Implementation (Advanced Features)
5. **PDF Manipulation MCP** - For complex document operations

## Tool Integration Architecture

```
Resume Input (PDF) → PDF Reader MCP → Resume Analyzer Agent
Job Posting (PDF/URL) → PDF Reader MCP → Job Requirements Extractor
Job Search → Career Site Job Listing API MCP → Job Discovery → Application Coordinator
Skills Data → File Management → Skills Matcher Agent
Optimized Content → Document Operations MCP → Document Formatter
Final Resume → File Management → Output Storage
```

## Risk Assessment and Mitigation

### High-Risk Tools
- **Career Site Job Listing API MCP**: Commercial API with usage costs
  - **Mitigation**: Monitor API usage and implement caching
  - **Testing**: Start with free tier to validate functionality

### Medium-Risk Tools
- **Document Operations MCP**: Python dependency complexity
  - **Mitigation**: Test thoroughly in isolated environment
  - **Fallback**: Use basic text output if formatting fails

### Low-Risk Tools
- **PDF Reader MCP**: Well-maintained, professional package
- **File Management**: Built-in Claude capabilities

## Success Criteria for Tool Integration
- [ ] PDF Reader successfully extracts text from sample resume
- [ ] Document Operations generates formatted Word/PDF output
- [ ] Career Site Job Listing API MCP retrieves job postings without errors
- [ ] All tools integrate with Claude Desktop configuration
- [ ] Error handling implemented for each tool failure scenario

## Next Steps
1. Install PDF Reader MCP and test with sample resume
2. Set up Document Operations MCP in development environment
3. Configure Career Site Job Listing API MCP with Apify API token
4. Create test scripts for each tool's core functionality
5. Document integration patterns for sub-agent communication


## MCP Configs
```json
{
  "mcpServers": {
    "pdf-reader-mcp": {
      "type": "stdio",
      "command": "npx",
      "args": [
        "@sylphlab/pdf-reader-mcp"
      ],
      "env": {}
    },
    "apify-jobs": {
      "command": "npx",
      "args": [
        "mcp-remote",
        "https://mcp.apify.com/sse?actors=fantastic-jobs/career-site-job-listing-api",
        "--header",
        "Authorization: Bearer <YOUR_APIFY_API_TOKEN>"
      ]
    }
  }
}
```


    "apify-jobs": {
      "type": "stdio",
      "command": "npx",
      "args": [
        "mcp-remote",
        "https://mcp.apify.com/sse?actors=fantastic-jobs/career-site-job-listing-api",
        "--header",
        "Authorization: Bearer $apify_token"
      ]
    },