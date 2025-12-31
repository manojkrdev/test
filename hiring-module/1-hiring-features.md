## Feature 1: Career Page \& Job Management

### Purpose

Public-facing portal where candidates discover and apply for positions, aggregating jobs from all platforms in one unified interface.

### Functional Specifications

**Job Display Module**

- **Grid/List view toggle**: Allow candidates to switch between card and list layouts
- **Advanced filtering**: By department, location, employment type (Full-time, Contract, Internship), experience level, posted date
- **Search functionality**: Full-text search across title, description, skills, and location
- **Job details page**: Comprehensive view with description, requirements, responsibilities, benefits, team info, similar jobs
- **Social sharing**: One-click sharing to LinkedIn, Twitter, WhatsApp with pre-filled text
- **Job alerts**: Candidates can subscribe to email alerts for matching new jobs

**Job Aggregation Logic**

```
SYNC PROCESS (runs every 15 minutes):
1. Fetch jobs from LinkedIn API → parse → standardize format
2. Fetch jobs from Indeed API → parse → standardize format
3. Fetch jobs from Naukri API → parse → standardize format
4. Check for duplicates
5. Create/update jobs table with external_job_ids
6. Update job_platforms table with sync status
7. Mark jobs not found in APIs as "Closed"
```

## Feature 2: Candidate Application Flow

### Purpose

Streamlined application process with resume parsing to minimize manual data entry and reduce drop-off rates.

### Flow Diagram

![Candidate application flow from job browsing to application submission with resume parsing and auto-fill](https://ppl-ai-code-interpreter-files.s3.amazonaws.com/web/direct-files/21f8d48b7e5dfce622fcb1ec85dabe90/72a4c904-45a3-4a98-b014-25da4d3ecadc/e66f2448.png)

Candidate application flow from job browsing to application submission with resume parsing and auto-fill

### Functional Specifications

**Resume Upload \& Parsing**

- **Supported formats**: PDF, DOC, DOCX, RTF, TXT (max 5MB)
- **Parsing API integration**: Use RChilli or Skima AI with 99% accuracy[^5][^6]
- **Extracted fields**: Personal info (name, email, phone), work experience (company, title, duration, description), education (degree, institution, year), skills (technical, soft skills), certifications, languages
- **Fallback handling**: If parsing fails, show empty form with manual entry option
- **Multi-resume support**: Allow candidates to upload multiple versions (technical resume, portfolio)

**Smart Form Auto-Fill**

- **Real-time preview**: Show parsed data before form population
- **Field mapping**: Map parsed JSON to form fields with confidence scores
- **Editable fields**: All auto-filled data remains editable
- **Validation**: Email format, phone number format (India: 10 digits), required field checks
- **Progressive disclosure**: Show optional sections (certifications, projects) only if parsed data exists

**Application Submission**

- **Duplicate detection**: Check if candidate email already applied to same job
- **Instant feedback**: Show success message with application ID (e.g., APP-2025-001234)
- **Confirmation email**: Send branded email with application summary, timeline, tracking link
- **Whatsapp notification**: Optional Whatsapp to candidate's phone with tracking URL

**Database Operations**

1. **Check candidate exists**: Query `candidates` by email
2. **Create/Update candidate**: Insert or update with parsed data
3. **Create application**: Insert into `applications` with status="Applied"
4. **Link documents**: Insert resume into `documents` table
5. **Trigger scoring**: Async call to Scoring Engine Service
6. **Create timeline**: Insert first entry into `application_stages`

**Error Handling**

- Parser timeout (>20s): Show manual form immediately
- Duplicate application: Notify candidate, offer to update existing application
- File upload failure: Retry with exponential backoff, show progress
- API rate limit: Queue request for processing, notify candidate via email

---

## Feature 3: HR Candidate Management Dashboard

### Purpose

Centralized interface for HR to review applications, update statuses, view ATS scores, and manage candidate pipeline.

### Functional Specifications

**Candidate List View**

- **Smart sorting**: Default sort by ATS score (highest first), then by application date
- **Filtering options**: By job, status (Applied, Reviewing, Interviewed, Selected, Rejected), date range, score range (>80%, 60-80%, <60%), source (Career Page, LinkedIn, Indeed, Naukri, Referral)
- **Bulk actions**: Select multiple candidates to move stage, send emails, export to CSV
- **Quick preview**: Hover card showing photo, key skills, experience, score without opening full profile
- **Tags**: Custom tags like "Urgent", "Top Talent", "Second Round", "On Hold"

**Candidate Detail Page**

- **Left panel**: Photo, contact info, social links, download resume button, application timeline
- **Center panel**: Tabbed interface
  - **Overview**: Summary with key highlights, ATS score breakdown, experience timeline
  - **Resume**: Embedded PDF viewer or parsed text display
  - **Documents**: List of all submitted documents with download/preview
  - **Activity**: All status changes, notes, emails sent, interview schedules
  - **Interview Feedback**: Ratings and comments from all interviewers
- **Right panel**: Quick actions
  - Status dropdown (Applied → Screening → Interview → Offer → Hired)
  - Schedule interview button
  - Request documents button
  - Add private notes (visible only to HR)
  - Reject with reason selection

**Status Management**

- **Predefined statuses**: Applied, Screening, Shortlisted, Interview Scheduled, Interview Completed, Offer Extended, Offer Accepted, Hired, Rejected, Withdrawn
- **Custom statuses**: Allow HR to add role-specific statuses per job
- **Automated transitions**: Moving to "Interview Scheduled" triggers calendar invite
- **Email triggers**: Each status change can trigger customizable email templates
- **Audit trail**: Track who changed status, when, and why

---

## Feature 4: ATS Scoring Engine

### Purpose

AI-powered matching algorithm that ranks candidates based on job requirements to prioritize top matches for HR review.

### Functional Specifications

**Scoring Algorithm Components**

**1. Skills Matching (40% weight)**

```
- Extract required skills from job description using JD parsing
- Extract candidate skills from parsed resume
- Calculate exact matches (case-insensitive)
- Calculate semantic matches (e.g., "React.js" matches "ReactJS")
- Use skill taxonomy (e.g., "Java" parent of "Spring Boot")
- Formula: (matched_skills / required_skills) × 40
```

**2. Experience Matching (30% weight)**

```
- Extract required experience from JD (e.g., "3-5 years")
- Calculate candidate's total experience from resume
- Scoring:
  - Exact match: 30 points
  - Within range: 25 points
  - 1 year below: 20 points
  - 2+ years below: 10 points
  - Over-qualified (5+ years above): 15 points
```

**3. Education Matching (15% weight)**

```
- Extract required degree from JD (e.g., "Bachelor's in Computer Science")
- Compare with candidate's education
- Scoring:
  - Exact match: 15 points
  - Related field: 12 points
  - Higher degree: 15 points
  - Lower degree: 8 points
  - No degree mentioned: 5 points
```

**4. Keyword \& Context Matching (15% weight)**

```
- Extract domain keywords from JD (technologies, methodologies, tools)
- Calculate TF-IDF scores for keywords in resume
- Bonus for industry-specific terms
- Bonus for company names matching job's industry
```

**Score Breakdown Display**

```
Overall Score: 87%
├── Skills: 36/40 (9/10 required skills matched)
├── Experience: 28/30 (4.5 years vs 3-5 required)
├── Education: 15/15 (Master's in CS vs Bachelor's required)
└── Keywords: 13/15 (High relevance for "microservices", "AWS", "Agile")
```

## Feature 5: Dynamic Interview Rounds Configuration

### Purpose

Flexible workflow system allowing HR to define custom interview stages per job with automated scheduling and feedback collection.

### Functional Specifications

**Round Configuration Interface**

- **Job template selection**: Choose from pre-defined templates (Engineering, Sales, Marketing, Leadership)
- **Custom rounds builder**: Drag-and-drop interface to add/reorder rounds
- **Round properties**:
  - Round name (e.g., "Technical Round 1", "Hiring Manager Discussion")
  - Round type (Phone Screen, Video Interview, In-person, Assignment, HR Interview)
  - Duration (30 min, 45 min, 1 hour, 2 hours)
  - Interviewers (assign users with email notifications)
  - Evaluation criteria (Technical Skills, Communication, Culture Fit, Problem Solving)
  - Auto-schedule or Manual scheduling
  - Required documents before this round

**Interview Scheduling**

- **Calendar integration**: Sync with Google Calendar, Outlook Calendar via APIs
- **Availability checking**: Show interviewer's free slots
- **Candidate self-scheduling**: Send link where candidate picks from available slots
- **Automated invites**: Send calendar invites to candidate and interviewers with meeting details
- **Video conference links**: Auto-generate Zoom/Google Meet/Teams links
- **Reminders**: Email reminders 24 hours and 1 hour before interview

**Feedback Collection**

- **Post-interview form**: Interviewers receive email with feedback form link
- **Rating system**: 5-star ratings on defined criteria
- **Structured questions**:
  - Technical competency: 1-5 rating + comments
  - Communication skills: 1-5 rating + comments
  - Culture fit: 1-5 rating + comments
  - Overall recommendation: Strong Yes, Yes, Maybe, No, Strong No
- **Aggregate scoring**: Calculate weighted average across all interviewers
- **HR visibility**: Dashboard showing pending feedback requests

## Feature 6: Document Request System

### Purpose

Progressive document collection workflow that requests additional documents from shortlisted candidates with tracking and reminders.

### Functional Specifications

**Document Types \& Configuration**

- **Pre-defined types**: Photo, ID Proof (Aadhaar, PAN, Passport), Address Proof, Educational Certificates, Experience Letters, Salary Slips (last 3 months), Relieving Letter, Background Verification Consent
- **Custom types**: HR can add role-specific documents (e.g., "Portfolio", "Code Samples", "Design Work")
- **Required vs Optional**: Mark documents as mandatory or optional
- **Stage-based requests**: Configure which documents needed at which stage (e.g., certificates needed before Offer)

**Request Workflow**

1. **HR triggers request**: From candidate detail page, select documents to request
2. **Candidate notification**: Email + SMS with secure upload link (token-based, expires in 7 days)
3. **Upload portal**: Candidate accesses personalized page showing requested documents with status
4. **File validation**: Check file type, size (<10MB), scan for malware
5. **Upload confirmation**: Email to candidate + notification to HR
6. **HR review**: HR marks document as "Approved", "Rejected (re-upload needed)", or "Under Review"
7. **Reminders**: Automated reminders every 2 days if documents pending

**Candidate Upload Portal**

```
┌─────────────────────────────────────────────┐
│ Documents Required for [Job Title]          │
├─────────────────────────────────────────────┤
│ ✓ Resume (Uploaded ✓)                       │
│ ⊗ Photo (Pending)          [Upload Button]  │
│ ⊗ ID Proof (Pending)       [Upload Button]  │
│ ✓ Degree Certificate (Under Review)         │
│ ⊗ Experience Letter (Rejected - Reupload)   │
│   Reason: Document not clear, please reupload│
└─────────────────────────────────────────────┘
Progress: 2/5 documents submitted
```

## Feature 7: Job Posting Platform (Multi-channel Publishing)

### Purpose

Single interface to create job postings and distribute them across career page, LinkedIn, Indeed, and Naukri with one click.

### Functional Specifications

**Job Creation Interface**

- **Smart JD builder**: Templates with pre-filled sections (About Company, Role Overview, Responsibilities, Requirements, Benefits)
- **AI-powered suggestions**: Suggest skills and requirements based on job title[^6]
- **Rich text editor**: Format job descriptions with bullets, headings, bold/italic
- **Field mapping**: Map internal fields to each platform's required format
- **Preview mode**: See how job will appear on each platform before publishing

**Platform-Specific Configurations**

**LinkedIn Jobs API**

- Required fields: Title, Description, Location, Employment Type, Experience Level, Job Function, Industry
- Optional: Salary range (improves visibility by 30% )[^7]
- Integration: OAuth 2.0 authentication → POST to `/jobs` endpoint
- Sync: Fetch applications via webhook when candidate applies on LinkedIn

**Indeed Publisher API**

- Required: Title, Description, Location, Job Type, How to Apply URL (your career page)
- Pricing: Cost-per-click or organic (free but lower visibility)
- Application flow: Indeed redirects to your career page for application

**Naukri RecruiterAPI**

- Required: Job Title, Description, Location, Functional Area, Industry, Key Skills, Min/Max Experience, Min/Max Salary
- Unique: Supports Indian cities, salary in INR, experience in years
- Application sync: Poll API every hour for new applications

**Multi-Post Workflow**

1. HR fills job form once with all common fields
2. System validates required fields for each selected platform
3. If missing platform-specific fields, show additional form
4. Click "Publish to All" button
5. Async job creates posts on each platform via APIs
6. Store external job IDs in `job_platforms` table
7. Show publishing status: "LinkedIn ✓, Indeed ✓, Naukri ⏳"
8. Send confirmation email to HR with links to each posting

### Technical Specifications

**API Integration Details**

**LinkedIn Talent Solutions**

```javascript
// Authentication
POST https://www.linkedin.com/oauth/v2/accessToken
Body: grant_type=client_credentials&client_id={}&client_secret={}

// Create Job
POST https://api.linkedin.com/v2/jobs
Headers: Authorization: Bearer {access_token}
Body: {
  "organization": "urn:li:organization:{company_id}",
  "title": "Senior Backend Developer",
  "description": "...",
  "location": "urn:li:location:us-sf-bay-area",
  "employmentType": "FULL_TIME",
  "externalJobPostingId": "{internal_job_id}"
}
```

**Indeed Publisher API**

```javascript
POST https://api.indeed.com/ads/api/v1/jobs
Headers: Authorization: Bearer {api_key}
Body: {
  "title": "Senior Backend Developer",
  "description": "...",
  "location": "San Francisco, CA",
  "jobType": "FULL_TIME",
  "applicationUrl": "https://yourcompany.com/careers/job-123"
}
```

**Error Handling \& Retry Logic**

- API timeout: Retry with exponential backoff (2s, 4s, 8s)
- Rate limit exceeded: Queue job for later, notify HR
- Invalid credentials: Alert admin via email, disable integration
- Partial success: If 2/3 platforms succeed, mark as published, retry failed one

**Job Sync \& Updates**

- **Update job**: When HR edits job, update on all platforms via PATCH APIs
- **Close job**: When HR closes job, mark as closed on all platforms
- **Fetch applications**: Scheduled job every 30 minutes fetches new applications from external platforms
- **Deduplication**: Match candidates by email to avoid duplicate profiles

---

## Feature 8: Employee Referral System

### Purpose

Empower employees to refer qualified candidates with a streamlined process, tracking, and rewards management.

### Functional Specifications

**Referral Submission Flow**

1. Employee logs into referral portal
2. Browse open jobs with search/filter (department, location, skills)
3. Click "Refer Candidate" button on job card
4. Fill quick form:
   - Candidate name, email, phone (required)
   - LinkedIn profile URL (optional but recommended)
   - Resume upload (optional, can apply later)
   - Relationship (Ex-colleague, Friend, LinkedIn connection, Family, Other)
   - Why you're referring (text area, max 500 chars)
5. Submit → Candidate receives email with application link
6. If resume uploaded: Auto-create candidate profile and application
7. If no resume: Send email asking candidate to complete application within 7 days

**Referral Dashboard (Employee View)**

```
┌────────────────────────────────────────────────┐
│ My Referrals                    Total: 12      │
├────────────────────────────────────────────────┤
│ [Active (5)] [Hired (2)] [Rejected (3)] [...]  │
├────────────────────────────────────────────────┤
│ Jane Doe - Senior Developer                    │
│ Referred: Dec 10, 2025                          │
│ Status: Interview Scheduled ⏰                  │
│ Next: Technical Round (Dec 30)                  │
├────────────────────────────────────────────────┤
│ John Smith - Product Manager                    │
│ Referred: Nov 22, 2025                          │
│ Status: Offer Accepted ✓                        │
│ Reward: ₹50,000 (Pending - after probation)    │
└────────────────────────────────────────────────┘
```

**Referral Tracking (HR View)**

- **Referral source report**: Which employees refer most candidates
- **Quality metrics**: Conversion rate (referrals to hires) per employee
- **Leaderboard**: Top 10 referrers with badges (Gold, Silver, Bronze)
- **Department breakdown**: Which departments have highest referral participation

**Rewards Management**

- **Reward tiers**: Configure rewards per role level (e.g., Engineer: ₹50k, Senior: ₹75k, Leadership: ₹1L)
- **Payout conditions**:
  - Application submitted (₹5k bonus)
  - Candidate hired (50% of reward)
  - Candidate completes probation (remaining 50%)
- **Approval workflow**: Finance approves payouts, integrates with payroll system
- **Tax handling**: Generate Form 16A for TDS compliance

## Feature 9: Application Status Tracking (Candidate Portal)

### Purpose

Transparent tracking interface allowing candidates to monitor their application progress similar to shipment tracking.

### Functional Specifications

**Tracking Portal Access**

- **Unique tracking link**: `yourcompany.com/track/APP-2025-001234` sent via email
- **Token-based authentication**: JWT token with application_id claim, expires in 90 days
- **Mobile-responsive**: Optimized for mobile viewing as 70% candidates check status on phones

**Status Timeline Display**

```
Application for: Senior Backend Developer

┌────────────────────────────────────┐
│ ● Applied           Dec 15, 2025   │ ✓
├────────────────────────────────────┤
│ ● Under Review      Dec 17, 2025   │ ✓
│   Your application is being         │
│   reviewed by our team               │
├────────────────────────────────────┤
│ ○ Interview         Expected: Week  │ ⏰
│   of Dec 25                          │
├────────────────────────────────────┤
│ ○ Decision          Expected: Jan 5 │
└────────────────────────────────────┘

Estimated time: 3-4 weeks
Next step: You'll hear from us in 5-7 days
```

**Status Types \& Candidate-Friendly Messaging**

- **Applied**: "We've received your application and will review it soon"
- **Under Review**: "Our team is reviewing your profile - average response time: 5 days"
- **Shortlisted**: "Great news! You've been shortlisted for the next round"
- **Interview Scheduled**: "Interview scheduled for [date] at [time] - [video link]"
- **Interview Completed**: "Thank you for interviewing with us. Decision expected in 3-5 days"
- **Offer Extended**: "Congratulations! We'd love you to join our team"
- **On Hold**: "Your application is on hold due to hiring freeze. We'll update you soon"
- **Rejected**: "Thank you for your interest. Unfortunately, we're moving forward with other candidates"

**Interactive Features**

- **Download documents**: Access to offer letter, interview schedule PDF
- **Upload documents**: If HR requested documents, upload directly from portal
- **Withdraw application**: Button to withdraw if candidate no longer interested
- **Feedback survey**: After rejection, optional survey asking about candidate experience
- **Similar jobs**: Show other open positions candidate might be interested in

## Feature 10: Settings \& Configuration Module

### Purpose

Centralized administration panel for system configuration, integrations, templates, and user management.

### Functional Specifications

**1. Integration Management**

- **API Credentials**: Secure form to enter/update API keys for LinkedIn, Indeed, Naukri, Resume Parser
- **Test connection**: Button to verify credentials work correctly
- **Enable/Disable**: Toggle to activate/deactivate each integration
- **Sync frequency**: Configure how often to sync jobs/applications (15 min, 30 min, 1 hour)
- **Webhook URLs**: Display webhook endpoints for external platforms to push data

**2. Email Template Editor**

- **Pre-built templates**: Application Confirmation, Interview Invite, Rejection, Offer Letter, Document Request, Referral Invitation
- **Visual editor**: WYSIWYG editor with variable insertion ({{candidate_name}}, {{job_title}}, {{company_name}})
- **Preview**: Send test email to check formatting
- **Multi-language**: Support for regional languages (Hindi, Tamil for India market)
- **Scheduling**: Configure send time (immediate vs scheduled)

**3. Career Page Customization**

- **Branding**: Upload logo, choose color scheme, add company banner image
- **About section**: Rich text editor for company description, culture, benefits
- **Contact info**: Email, phone, social media links
- **SEO settings**: Meta title, description, keywords
- **Custom domain**: Map custom domain (careers.yourcompany.com)

**4. ATS Scoring Weights**

- **Adjust algorithm**: Slider to change weights (Skills: 30-50%, Experience: 20-40%, etc.)
- **Custom criteria**: Add company-specific criteria (e.g., "Startup Experience": 10%)
- **Preview impact**: Show how weight changes affect sample candidate scores
- **Per-job override**: Allow different scoring for different job types
