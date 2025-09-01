# S.A.R.D. Web App - MVP Specification

**Project:** Secure AI-driven R&D Project Analyzer - Web Application  
**Version:** MVP 1.0  
**Date:** 2025-09-01  
**Stage:** MVP (as per .windsurfrules)

## Executive Summary

Transform the existing Python-based S.A.R.D. analyzer into a modern web application that enables users to upload, analyze, and evaluate startup documents through an intuitive interface. The MVP focuses on core document processing and startup evaluation capabilities with a clean, responsive UI.

## Core Value Proposition

- **Document Analysis**: Upload PDF, DOCX, PPTX files for automated parsing and classification
- **Startup Evaluation**: Comprehensive scoring system for business ideas, market analysis, team assessment
- **Secure Processing**: Local LLM for classified documents, remote LLM for unclassified content
- **Report Generation**: Professional evaluation reports with scoring metrics

## Target Users

- **Primary**: Startup accelerators, incubators, and investment firms
- **Secondary**: Business analysts, venture capitalists, startup founders

## MVP Feature Set

### 1. Document Upload & Management
- **File Upload**: Drag-and-drop interface for PDF, DOCX, PPTX files
- **File Validation**: Size limits (50MB), format checking, virus scanning
- **Document Library**: List view of uploaded documents with metadata
- **Batch Processing**: Upload multiple files simultaneously

### 2. Document Analysis Engine
- **Text Extraction**: Parse content from supported file formats
- **Classification**: Automatic classification (classified/unclassified) with manual override
- **Content Summarization**: AI-powered summaries using appropriate LLM based on classification
- **Progress Tracking**: Real-time processing status updates

### 3. Startup Evaluation System
- **Scoring Framework**: 6-category evaluation system (Business Idea, Market, Business Plan, Team, Financing, Pitch Deck)
- **Automated Analysis**: Keyword-based scoring with AI enhancement
- **Company Matching**: Link applications with corresponding pitch decks
- **Evaluation Reports**: Professional PDF/MD reports with detailed scoring

### 4. Results Dashboard
- **Company Overview**: Grid/list view of all evaluated companies
- **Score Visualization**: Charts and graphs showing evaluation metrics
- **Report Management**: Download, share, and archive evaluation reports
- **Search & Filter**: Find companies by name, score range, or evaluation date

### 5. User Management (Basic)
- **Authentication**: Simple email/password login
- **Session Management**: Secure session handling
- **User Preferences**: Basic settings for LLM mode, report format

## Technical Architecture

### Frontend Stack
- **Framework**: Next.js 15.1.3 with React 19.0.0
- **Styling**: Tailwind CSS 3.4.17 + shadcn/ui 2.1.8
- **State Management**: React Context + useReducer
- **File Handling**: React Dropzone for uploads
- **Charts**: Recharts for data visualization

### Backend Stack
- **Runtime**: Node.js 20.0.0 with TypeScript 5.0.0
- **Database**: SQLite with Prisma 5.0.0 ORM
- **File Processing**: 
  - PDF: pdf-parse or pdf2pic
  - DOCX: mammoth.js
  - PPTX: officegen or similar
- **LLM Integration**: 
  - OpenAI API for remote processing
  - Ollama HTTP API for local processing

### Database Schema

```sql
-- Users table
CREATE TABLE users (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  email TEXT UNIQUE NOT NULL,
  password_hash TEXT NOT NULL,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- Documents table (enhanced from existing)
CREATE TABLE documents (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  user_id INTEGER REFERENCES users(id),
  filename TEXT NOT NULL,
  original_name TEXT NOT NULL,
  file_path TEXT NOT NULL,
  file_size INTEGER NOT NULL,
  raw_text TEXT,
  classification TEXT,
  summary TEXT,
  source_type TEXT,
  date_added DATETIME DEFAULT CURRENT_TIMESTAMP,
  processing_status TEXT DEFAULT 'pending'
);

-- Companies table
CREATE TABLE companies (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  user_id INTEGER REFERENCES users(id),
  name TEXT NOT NULL,
  application_doc_id INTEGER REFERENCES documents(id),
  pitch_deck_doc_id INTEGER REFERENCES documents(id),
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- Evaluations table
CREATE TABLE evaluations (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  company_id INTEGER REFERENCES companies(id),
  business_idea_score INTEGER,
  market_score INTEGER,
  business_plan_score INTEGER,
  team_score INTEGER,
  financing_score INTEGER,
  pitch_deck_score INTEGER,
  total_score INTEGER,
  evaluation_summary TEXT,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

## API Endpoints

### Authentication
- `POST /api/auth/login` - User login
- `POST /api/auth/logout` - User logout
- `GET /api/auth/me` - Get current user

### Documents
- `POST /api/documents/upload` - Upload single/multiple files
- `GET /api/documents` - List user's documents
- `GET /api/documents/:id` - Get document details
- `DELETE /api/documents/:id` - Delete document
- `POST /api/documents/:id/process` - Process document (parse + classify + summarize)

### Companies
- `GET /api/companies` - List companies with evaluations
- `POST /api/companies` - Create company (link documents)
- `GET /api/companies/:id` - Get company details
- `PUT /api/companies/:id` - Update company
- `DELETE /api/companies/:id` - Delete company

### Evaluations
- `POST /api/companies/:id/evaluate` - Run evaluation for company
- `GET /api/evaluations/:id` - Get evaluation details
- `GET /api/evaluations/:id/report` - Download evaluation report

## User Interface Design

### Layout Structure
```
Header: Logo, Navigation, User Menu
Sidebar: Quick Actions, Recent Documents, Settings
Main Content: Dynamic based on current page
Footer: Status, Version, Links
```

### Key Pages

#### 1. Dashboard (`/dashboard`)
- Upload area (prominent)
- Recent documents list
- Quick stats (total docs, companies evaluated)
- Recent evaluations preview

#### 2. Documents (`/documents`)
- File upload zone
- Documents table with filters
- Processing status indicators
- Bulk actions (delete, process)

#### 3. Companies (`/companies`)
- Company cards/table view
- Score visualization
- Filter by score range, date
- Create new company button

#### 4. Company Detail (`/companies/:id`)
- Company overview
- Document previews
- Evaluation scores with breakdown
- Report download options

#### 5. Settings (`/settings`)
- LLM mode selection
- File processing preferences
- Account settings

## Development Phases

### Phase 1: Core Infrastructure (Week 1-2)
- [ ] Next.js project setup with TypeScript
- [ ] Database schema implementation with Prisma
- [ ] Basic authentication system
- [ ] File upload functionality
- [ ] Document parsing integration

### Phase 2: Analysis Engine (Week 3-4)
- [ ] Document classification logic
- [ ] LLM integration (OpenAI + Ollama)
- [ ] Summarization pipeline
- [ ] Background job processing

### Phase 3: Evaluation System (Week 5-6)
- [ ] Startup evaluation algorithm
- [ ] Company management interface
- [ ] Scoring visualization
- [ ] Report generation

### Phase 4: UI/UX Polish (Week 7-8)
- [ ] Responsive design implementation
- [ ] Dashboard with analytics
- [ ] User experience optimization
- [ ] Error handling and loading states

## Technical Requirements

### Performance
- File upload: Support up to 50MB files
- Processing: Handle 10 documents simultaneously
- Response time: API calls < 2 seconds (excluding LLM calls)
- UI responsiveness: < 100ms for interactions

### Security
- File validation and sanitization
- Secure file storage with access controls
- API rate limiting
- Input validation and XSS protection
- Secure session management

### Scalability Considerations
- Database indexing for search performance
- File storage optimization
- Background job queue for processing
- Caching for frequently accessed data

## Environment Configuration

### Required Environment Variables
```env
# Database
DATABASE_URL="file:./dev.db"

# OpenAI
OPENAI_API_KEY="sk-..."

# Ollama
OLLAMA_HOST="http://localhost:11434"

# File Storage
UPLOAD_DIR="./uploads"
MAX_FILE_SIZE="52428800"

# Authentication
JWT_SECRET="your-secret-key"
SESSION_SECRET="your-session-secret"
```

### Development Setup
1. Clone repository
2. Install dependencies: `npm install`
3. Setup database: `npx prisma migrate dev`
4. Configure environment variables
5. Start development server: `npm run dev`

## Success Metrics

### MVP Success Criteria
- [ ] Successfully process 100+ documents without errors
- [ ] Generate evaluation reports for 20+ companies
- [ ] User can complete full workflow in < 10 minutes
- [ ] 95% uptime during testing period
- [ ] Positive feedback from 3+ test users

### Key Performance Indicators
- Document processing success rate: >95%
- Average evaluation completion time: <5 minutes
- User retention after first use: >70%
- Report generation accuracy: Manual verification of 10 samples

## Risk Mitigation

### Technical Risks
- **LLM API failures**: Implement retry logic and fallback mechanisms
- **File processing errors**: Comprehensive error handling and user feedback
- **Performance bottlenecks**: Database optimization and caching strategies

### Business Risks
- **User adoption**: Focus on clear value proposition and smooth onboarding
- **Data security**: Implement proper encryption and access controls
- **Scalability**: Design with growth in mind, use cloud-ready architecture

## Future Enhancements (Post-MVP)

### Phase 2 Features
- Advanced analytics and reporting
- Team collaboration features
- Integration with external data sources
- Mobile-responsive design improvements

### Phase 3 Features
- Machine learning model training
- API for third-party integrations
- Advanced user management and permissions
- Real-time collaboration features

---

**Next Steps:**
1. Review and approve this specification
2. Set up development environment
3. Begin Phase 1 implementation
4. Establish testing and deployment pipeline

*This specification follows MVP stage requirements with focus on core functionality, basic testing, and user validation.*
