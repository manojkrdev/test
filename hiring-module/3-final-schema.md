## Authentication \& User Management

### 1. m_guest_users

**Purpose:** Handles OTP-based authentication for candidates applying to jobs. Keeps guest users separate from internal employees for security.

**Key Features:**

- OTP verification with attempt limiting
- Token versioning for session invalidation
- Account locking mechanism for security
- Tracks last login and IP for monitoring

```sql
CREATE TABLE m_guest_users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR(255) UNIQUE NOT NULL,
  
  otp_code_hash VARCHAR(255),
  otp_expires_at TIMESTAMP,
  otp_attempts INT DEFAULT 0,
  otp_last_sent_at TIMESTAMP,
  
  token_version INT DEFAULT 1,
  
  last_ip_address INET,
  last_login_at TIMESTAMP,
  
  failed_login_attempts INT DEFAULT 0,
  locked_until TIMESTAMP,
  is_active BOOLEAN DEFAULT true,
  
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```


***

### 2. m_candidates

**Purpose:** Stores complete candidate profile information including resume data, skills, experience, and preferences. Central repository for all candidate-related data.

**Key Features:**

- Links to guest_user for authentication
- Resume parsing data storage (JSONB)
- Dual skill storage (JSONB + array) for flexibility and search performance
- Profile completion tracking
- Current and expected CTC for matching

```sql
CREATE TABLE m_candidates (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  guest_user_id UUID UNIQUE,
  email VARCHAR(255) UNIQUE NOT NULL,
  
  full_name VARCHAR(255) NOT NULL,
  phone VARCHAR(20) NOT NULL,
  alternate_phone VARCHAR(20),
  preferred_contact_method VARCHAR(50) DEFAULT 'email',
  
  current_location VARCHAR(255),
  current_city VARCHAR(100),
  
  total_experience_years DECIMAL(4,1),
  current_employment_status VARCHAR(50),
  current_company VARCHAR(255),
  current_designation VARCHAR(255),
  notice_period_days INT,
  current_ctc DECIMAL(12,2),
  expected_ctc DECIMAL(12,2),
  
  linkedin_profile TEXT,
  portfolio_links JSONB,
  
  key_skills JSONB NOT NULL,
  primary_skills TEXT[],
  
  languages JSONB,
  willing_to_relocate BOOLEAN DEFAULT false,
  preferred_work_mode VARCHAR(50),
  
  education_history JSONB NOT NULL,
  highest_education VARCHAR(100),
  work_experience JSONB,
  
  resume_link TEXT,
  resume_parsed_data JSONB,
  
  profile_completion_percentage INT DEFAULT 0,
  is_active BOOLEAN DEFAULT true,
  
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```


***

### 3. t_candidate_to_employee

**Purpose:** Manages the transition of hired candidates into internal employees. Tracks onboarding progress and bridges recruitment and HR systems.

**Key Features:**

- One-to-one mapping (candidate → employee)
- Onboarding checklist (documents, background check, system access)
- Joining date tracking (expected vs actual)
- Status workflow from offer acceptance to active employee

```sql
CREATE TABLE t_candidate_to_employee (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  
  candidate_id UUID NOT NULL UNIQUE,
  application_id UUID NOT NULL UNIQUE,
  offer_id UUID NOT NULL UNIQUE,
  
  employee_id UUID UNIQUE,
  employee_code VARCHAR(50) UNIQUE,
  
  department_id UUID NOT NULL,
  designation_id UUID NOT NULL,
  reporting_manager_id UUID,
  work_location VARCHAR(255),
  
  expected_joining_date DATE NOT NULL,
  actual_joining_date DATE,
  
  status VARCHAR(50) DEFAULT 'offer_accepted',
  
  background_verification_status VARCHAR(50) DEFAULT 'pending',
  documents_verified BOOLEAN DEFAULT false,
  system_access_created BOOLEAN DEFAULT false,
  
  initiated_by UUID,
  initiated_at TIMESTAMP,
  notes TEXT,
  
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```


***

### 4. t_security_logs

**Purpose:** Records all security-related events for auditing, debugging, and threat detection. Tracks authentication attempts, suspicious activities, and important actions.

**Key Features:**

- Tracks guest users, candidates, and internal users
- Event classification (type, status, severity)
- IP and endpoint tracking
- Suspicious activity flagging

```sql
CREATE TABLE t_security_logs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  
  user_type VARCHAR(50) NOT NULL,
  user_id UUID,
  email VARCHAR(255),
  
  event_type VARCHAR(100) NOT NULL,
  event_status VARCHAR(50) NOT NULL,
  severity VARCHAR(20) DEFAULT 'info',
  
  entity_type VARCHAR(50),
  entity_id UUID,
  
  ip_address INET,
  user_agent TEXT,
  request_endpoint VARCHAR(255),
  
  is_suspicious BOOLEAN DEFAULT false,
  security_notes TEXT,
  
  metadata JSONB,
  
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```


***

## Job Management

### 5. m_jobs

**Purpose:** Stores job postings with requirements, descriptions, and compensation details. Central table for all job-related information displayed on career page.

**Key Features:**

- Experience range and education requirements
- Required and preferred skills (JSONB)
- Salary range with display control
- Draft/active/closed status workflow
- View and application counters

```sql
CREATE TABLE m_jobs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  job_code VARCHAR(50) UNIQUE NOT NULL,
  job_title VARCHAR(255) NOT NULL,
  department_id UUID,
  
  -- Location & Work
  work_location VARCHAR(255) NOT NULL,
  work_mode VARCHAR(50) NOT NULL,
  employment_type VARCHAR(50) NOT NULL,
  openings_count INT DEFAULT 1,
  
  -- Experience & Education
  min_experience_years DECIMAL(3,1) NOT NULL,
  max_experience_years DECIMAL(3,1) NOT NULL,
  minimum_education VARCHAR(100),
  
  -- Skills
  required_skills JSONB NOT NULL,
  preferred_skills JSONB,
  
  -- Description
  job_summary TEXT NOT NULL,
  responsibilities JSONB NOT NULL,
  detailed_description TEXT,
  
  -- Compensation
  display_salary BOOLEAN DEFAULT false,
  min_salary DECIMAL(12,2),
  max_salary DECIMAL(12,2),
  currency VARCHAR(3) DEFAULT 'INR',
  
  -- Settings
  application_deadline DATE,
  max_notice_period_days INT,
  screening_questions JSONB,
  
  -- Status
  status VARCHAR(50) DEFAULT 'draft',
  published_at TIMESTAMP,
  closed_at TIMESTAMP,
  
  -- Stats
  view_count INT DEFAULT 0,
  application_count INT DEFAULT 0,
  
  created_by UUID,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```


***

### 6. t_job_platforms

**Purpose:** Tracks where each job is published (LinkedIn, Indeed, Naukri, career page). Enables multi-platform job distribution and syncing.

**Key Features:**

- Stores external platform job IDs
- Publication status tracking
- Direct links to external postings
- Supports application source tracking

```sql
CREATE TABLE t_job_platforms (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  job_id UUID NOT NULL,
  platform_name VARCHAR(50) NOT NULL,
  external_job_id VARCHAR(255),
  post_url TEXT,
  is_published BOOLEAN DEFAULT false,
  published_at TIMESTAMP,
  
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```


***

## Application Management

### 7. t_applications

**Purpose:** Records candidate job applications with status, scoring, and assignment details. Core table for application tracking and management.

**Key Features:**

- ATS scoring with breakdown
- Source tracking (career page, LinkedIn, referral)
- Status workflow (applied → hired/rejected)
- HR assignment and priority levels
- Screening question answers

```sql
CREATE TABLE t_applications (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  application_code VARCHAR(50) UNIQUE NOT NULL,
  job_id UUID NOT NULL,
  candidate_id UUID NOT NULL,
  
  source VARCHAR(50) NOT NULL,
  referral_id UUID,
  
  expected_ctc DECIMAL(12,2) NOT NULL,
  earliest_joining_date DATE NOT NULL,
  screening_answers JSONB,
  
  status VARCHAR(50) DEFAULT 'applied',
  ats_score DECIMAL(5,2),
  ats_score_breakdown JSONB,
  
  assigned_to UUID,
  priority VARCHAR(20) DEFAULT 'normal',
  
  applied_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```


***

### 8. t_application_stages

**Purpose:** Maintains application timeline and stage history. Tracks every status change with notes and candidate communications.

**Key Features:**

- Chronological stage tracking
- Internal HR notes
- Candidate-facing messages
- Changed by user tracking

```sql
CREATE TABLE t_application_stages (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  application_id UUID NOT NULL,
  
  stage_name VARCHAR(100) NOT NULL,
  status VARCHAR(50) NOT NULL,
  
  notes TEXT,
  candidate_message TEXT,
  
  changed_by UUID,
  changed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```


***

## Interview Management

### 9. t_interviews

**Purpose:** Schedules and manages interview rounds for candidates. Stores meeting details, interviewers, and status.

**Key Features:**

- Round information (number, name, type)
- Video/phone/in-person meeting details
- Multiple interviewer support (JSONB)
- Cancellation and rescheduling tracking

```sql
CREATE TABLE t_interviews (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  application_id UUID NOT NULL,
  candidate_id UUID NOT NULL,
  
  -- Round Info
  round_number INT NOT NULL,
  round_name VARCHAR(100) NOT NULL,
  round_type VARCHAR(50) NOT NULL,
  
  -- Schedule
  scheduled_at TIMESTAMP NOT NULL,
  duration_minutes INT NOT NULL,
  
  -- Meeting
  meeting_link TEXT,
  meeting_password VARCHAR(100),
  location VARCHAR(255),
  
  -- Participants
  interviewer_ids JSONB NOT NULL,
  
  -- Status
  status VARCHAR(50) DEFAULT 'scheduled',
  cancellation_reason TEXT,
  
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```


***

### 10. t_interview_feedback

**Purpose:** Collects structured feedback from interviewers post-interview. Used for hiring decisions and candidate evaluation.

**Key Features:**

- Multiple rating dimensions (technical, communication, culture fit)
- Overall rating and recommendation
- Detailed comments section
- Links to specific interview and interviewer

```sql
CREATE TABLE t_interview_feedback (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  interview_id UUID NOT NULL,
  interviewer_id UUID NOT NULL,
  
  overall_rating DECIMAL(2,1) NOT NULL,
  recommendation VARCHAR(50) NOT NULL,
  
  technical_rating DECIMAL(2,1),
  communication_rating DECIMAL(2,1),
  culture_fit_rating DECIMAL(2,1),
  
  comments TEXT,
  
  submitted_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```


***

## Document Management

### 11. t_documents

**Purpose:** Stores all candidate-uploaded documents (resumes, certificates, ID proofs). Supports versioning and approval workflow.

**Key Features:**

- Multiple document types support
- Version control for resumes
- Primary document flagging
- HR review and approval status

```sql
CREATE TABLE t_documents (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  candidate_id UUID,
  application_id UUID,
  
  document_type VARCHAR(100) NOT NULL,
  file_name VARCHAR(255) NOT NULL,
  file_path TEXT NOT NULL,
  file_size INT,
  mime_type VARCHAR(100),
  
  version INT DEFAULT 1,
  is_primary BOOLEAN DEFAULT false,
  
  status VARCHAR(50) DEFAULT 'pending',
  reviewed_by UUID,
  reviewed_at TIMESTAMP,
  
  uploaded_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```


***

### 12. t_document_requests

**Purpose:** Manages HR requests for additional documents from candidates. Tracks completion and sends reminders.

**Key Features:**

- Secure token-based upload links
- Document status tracking per request
- Automated reminder system
- Expiring upload tokens

```sql
CREATE TABLE t_document_requests (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  application_id UUID NOT NULL,
  
  requested_documents JSONB NOT NULL,
  request_token VARCHAR(255) UNIQUE,
  token_expires_at TIMESTAMP,
  
  requested_by UUID,
  requested_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  completed_at TIMESTAMP,
  
  reminder_sent_count INT DEFAULT 0,
  last_reminder_at TIMESTAMP
);
```


***

## Offer \& Referral Management

### 13. t_offers

**Purpose:** Creates and manages job offers for selected candidates. Tracks offer details, approval, and candidate response.

**Key Features:**

- Comprehensive compensation details
- Offer validity tracking
- Approval workflow
- Status tracking (draft → sent → accepted/rejected)
- One offer per application (UNIQUE constraint)

```sql
CREATE TABLE t_offers (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  application_id UUID NOT NULL UNIQUE,
  candidate_id UUID NOT NULL,
  job_id UUID NOT NULL,
  
  offer_code VARCHAR(50) UNIQUE NOT NULL,
  
  annual_salary DECIMAL(12,2) NOT NULL,
  currency VARCHAR(3) DEFAULT 'INR',
  variable_component DECIMAL(12,2),
  joining_bonus DECIMAL(10,2),
  benefits JSONB,
  
  employment_type VARCHAR(50) NOT NULL,
  work_location VARCHAR(255) NOT NULL,
  work_mode VARCHAR(50) NOT NULL,
  probation_period_months INT DEFAULT 6,
  
  expected_joining_date DATE NOT NULL,
  offer_valid_until DATE NOT NULL,
  
  status VARCHAR(50) DEFAULT 'draft',
  sent_at TIMESTAMP,
  responded_at TIMESTAMP,
  
  approved_by UUID,
  approved_at TIMESTAMP,
  
  created_by UUID,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```


***

### 14. t_referrals

**Purpose:** Manages employee referral program. Tracks referred candidates and reward payouts.

**Key Features:**

- Links referrer to candidate
- Referral status tracking
- Reward amount and payout status
- Relationship and reason tracking

```sql
CREATE TABLE t_referrals (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  job_id UUID NOT NULL,
  referred_by UUID NOT NULL,
  
  candidate_name VARCHAR(255) NOT NULL,
  candidate_email VARCHAR(255) NOT NULL,
  candidate_phone VARCHAR(20),
  relationship VARCHAR(100),
  referral_reason TEXT,
  
  status VARCHAR(50) DEFAULT 'pending',
  application_id UUID,
  candidate_id UUID,
  
  reward_amount DECIMAL(10,2),
  reward_status VARCHAR(50) DEFAULT 'pending',
  reward_paid_at TIMESTAMP,
  
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```


***

## Communication Management

### 15. t_communication_logs

**Purpose:** Records all communications sent to candidates (email, SMS, WhatsApp). Provides audit trail and delivery tracking.

**Key Features:**

- Multi-channel support (email/SMS/WhatsApp)
- Template linkage
- Delivery status tracking
- Entity-based organization (application, interview, offer)

```sql
CREATE TABLE t_communication_logs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  
  entity_type VARCHAR(50) NOT NULL,
  entity_id UUID NOT NULL,
  candidate_id UUID,
  
  communication_type VARCHAR(50) NOT NULL,
  template_id UUID,
  
  recipient_email VARCHAR(255),
  recipient_phone VARCHAR(20),
  subject VARCHAR(500),
  body TEXT,
  
  status VARCHAR(50) DEFAULT 'pending',
  sent_at TIMESTAMP,
  
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```


***

### 16. m_email_templates

**Purpose:** Stores reusable email templates for candidate communications. Supports variable substitution and categorization.

**Key Features:**

- HTML and plain text support
- Dynamic variable support
- Category-based organization
- Active/inactive toggle

```sql
CREATE TABLE m_email_templates (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  template_name VARCHAR(255) NOT NULL,
  template_code VARCHAR(100) UNIQUE NOT NULL,
  
  subject VARCHAR(500) NOT NULL,
  body_html TEXT NOT NULL,
  
  available_variables JSONB,
  category VARCHAR(50),
  is_active BOOLEAN DEFAULT true,
  
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```


***

## System Configuration

### 17. m_system_settings

**Purpose:** Stores application-wide configuration settings. Centralized configuration management.

**Key Features:**

- Key-value storage
- Description for documentation
- Simple text-based values

```sql
CREATE TABLE m_system_settings (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  setting_key VARCHAR(100) UNIQUE NOT NULL,
  setting_value TEXT NOT NULL,
  description TEXT,
  
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```


***

### 18. m_integration_credentials

**Purpose:** Securely stores API credentials for external platforms (LinkedIn, Indeed, Naukri). Enables job posting integrations.

**Key Features:**

- Platform-specific credentials
- Active/inactive toggle
- Last sync tracking
- Secure credential storage

```sql
CREATE TABLE m_integration_credentials (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  platform_name VARCHAR(50) UNIQUE NOT NULL,
  
  api_key VARCHAR(500),
  api_secret VARCHAR(500),
  access_token VARCHAR(500),
  
  is_active BOOLEAN DEFAULT true,
  last_sync_at TIMESTAMP,
  
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```


***

### 19. m_career_page_settings

**Purpose:** Configures career page branding and content. Controls public-facing job portal appearance.

**Key Features:**

- Company branding (logo, colors)
- About section content
- Contact information
- Social media links

```sql
CREATE TABLE m_career_page_settings (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  company_name VARCHAR(255),
  company_logo_url TEXT,
  banner_image_url TEXT,
  primary_color VARCHAR(7),
  
  about_company TEXT,
  benefits_overview TEXT,
  
  contact_email VARCHAR(255),
  contact_phone VARCHAR(20),
  social_links JSONB,
  
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```


***

### 20. t_audit_logs

**Purpose:** Comprehensive audit trail for all entity changes. Tracks who changed what and when across the entire system.

**Key Features:**

- Entity-agnostic logging
- Before/after value tracking
- User type identification (user/system/candidate)
- IP address tracking

```sql
CREATE TABLE t_audit_logs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  
  entity_type VARCHAR(50) NOT NULL,
  entity_id UUID NOT NULL,
  action VARCHAR(100) NOT NULL,
  
  changed_by UUID,
  changed_by_type VARCHAR(50),
  
  old_values JSONB,
  new_values JSONB,
  
  ip_address INET,
  
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```


