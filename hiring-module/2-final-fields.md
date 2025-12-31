
# **A. JOB POSTING FORM (HR/Recruiter)**

## **SECTION 1: Job Basics** *(2 minutes)*

### **Required Fields**

- `job_code` (VARCHAR 50) - Auto-generated if empty: "DEPT-2025-001"
- `job_title` (VARCHAR 255) *
- `department_id` (VARCHAR 100) *
    - Engineering, Product, Sales, Marketing, Operations, HR, Finance, Design, Customer Success, Academic/Counseling, Admin, Legal
- `department_name` (VARCHAR 100) *
    - Engineering, Product, Sales, Marketing, Operations, HR, Finance, Design, Customer Success, Academic/Counseling, Admin, Legal
- `job_function` (VARCHAR 100) * - Auto-suggests based on department
    - **Tech:** Development, DevOps, QA/Testing, Data Science, Security
    - **Sales:** B2B Sales, B2C Sales, Inside Sales, Field Sales
    - **Counseling:** Academic, Career, Admissions, Student Support
    - **Operations:** Supply Chain, Logistics, Project Management
- `primary_location` (VARCHAR 255) *
- `alternate_location` (jsonb)
- `work_mode` (VARCHAR 50) *
    - Remote, Hybrid, On-site, Field-based
- `employment_type` (VARCHAR 50) *
    - Full-time, Part-time, Contract, Internship
- `openings_count` (INT) * - Default: 1
- `contract_duration_months` (INT) - Shows only if Contract/Internship selected

***

## **SECTION 2: Experience \& Education** *(1 minute)*

### **Required Fields**

- `experience_level` (VARCHAR 50) *
    - Fresher (0-1 yrs), Entry (1-3 yrs), Mid (3-6 yrs), Senior (6-10 yrs), Lead (10+ yrs)
- `min_experience_years` (INT) *
- `max_experience_years` (INT) *
- `minimum_education` (VARCHAR 100) *
    - Not Required, High School, Any Graduate, Bachelor's (any field), Bachelor's (specific field), Master's, PhD
- `preferred_education` (JSONB)
    - ["MBA", "M.Tech"]

***

## **SECTION 3: Skills \& Requirements** *(2 minutes)*

### **Domain-Adaptive Fields** (Show based on department)

**For Tech/Engineering:**

- `required_skills` (JSONB) * - Min 3, Max 10
    - ["Node.js", "React", "PostgreSQL", "AWS"]
    - 💡 **Auto-suggest** based on job title
- `preferred_skills` (JSONB)
    - ["GraphQL", "Kubernetes", "TypeScript"]
- `language_requirements` (JSONB) *

```json
[
  {"language": "English", "level": "Fluent"},
  {"language": "Hindi", "level": "Conversational"}
]
```

- `industry_experience` (JSONB) - For experienced roles
    - ["EdTech", "SaaS", "FinTech"]

***

## **SECTION 4: Job Description** *(3 minutes - with AI assistance)*

### **Required Fields**

- `job_summary` (TEXT) * - Max 250 chars
    - **Tips:** 2-3 sentences, highlight what's exciting about the role
    - **AI Helper:** "✨ Generate summary from responsibilities"
- `responsibilities` (JSONB) * - Min 4, Max 12 bullets

```json
[
  "Design and implement scalable backend APIs",
  "Lead code reviews and mentor junior developers",
  "Collaborate with product and design teams",
  "Optimize database queries for performance"
]
```

- `detailed_description` (TEXT) - Rich text editor
    - **Auto-generates** from summary + responsibilities if left empty
    - **AI Helper:** "✨ Expand into full description"
- `what_makes_this_role_special` (TEXT) - Max 500 chars
    - Highlight unique aspects: "Work with cutting-edge tech", "Direct impact on 1M+ students"
- `day_in_life` (TEXT)
    - "Typical day: Morning standup → Feature development → Code review → Team collaboration"

***

## **SECTION 5: Compensation \& Benefits** *(2 minutes)*

### **Salary Configuration**

- `display_salary` (BOOLEAN) * - Default: false
    - ⚠️ **Note:** Showing salary increases applications by 30-40%

**If display_salary = true:**

- `salary_type` (VARCHAR 50) *
    - Fixed, Fixed + Variable (for sales), Hourly
- `min_salary` (DECIMAL 12,2) *
- `max_salary` (DECIMAL 12,2) *
- `currency` (VARCHAR 3) - Default: INR
- `salary_period` (VARCHAR 20) - Default: annual
- `variable_component_percent` (INT) - "Up to 30% variable"
- `ote_description` (TEXT) - "On-target earnings: ₹15L (₹10L fixed + ₹5L variable)"


### **Benefits (Quick Checkboxes)**

- `benefits` (JSONB) - Select all that apply

```json
{
  "standard": ["Health Insurance", "Life Insurance", "Paid Leave"],
  "workplace": ["WFH", "Flexible Hours", "Hybrid Model"],
  "financial": ["Performance Bonus", "ESOPs", "Joining Bonus"],
  "learning": ["L&D Budget", "Certifications", "Conferences"],
  "lifestyle": ["Gym", "Meals", "Transport Allowance"]
}
```


### **Optional**

- `joining_bonus` (DECIMAL 10,2)
- `relocation_support` (BOOLEAN)
- `equipment_provided` (TEXT) - "Laptop, monitor, headphones"

***

## **SECTION 6: Application Settings** *(1 minute)*

### **Required Fields**

- `application_deadline` (DATE) - "Apply by"
- `expected_joining_date` (DATE) - "Start from"
- `notice_period_acceptable` (VARCHAR 50) *
    - Immediate, Up to 30 days, Up to 60 days, Up to 90 days, Flexible
- `interview_rounds` (INT) - Default: 3
- `expected_timeline` (VARCHAR 100)
    - "We'll get back in 5-7 days"
- `screening_questions` (JSONB) - Custom questions for candidates

```json
[
  {
    "question": "Why do you want to join us?",
    "type": "text",
    "required": true,
    "max_length": 500
  },
  {
    "question": "Available to join in 30 days?",
    "type": "yes_no",
    "required": true
  }
]
```


***

## **SECTION 7: Publishing** *(1 minute)*

### **Required Fields**

- `publish_immediately` (BOOLEAN) * - Default: true
    - If false: `scheduled_publish_at` (TIMESTAMP) or manual publish
- `publish_to_indeed` (BOOLEAN)
- `publish_to_naukri` (BOOLEAN)
- `publish_to_linkedin` (BOOLEAN) - Track leads only

**Conditional: If Naukri selected**

- `naukri_functional_area` (VARCHAR 100) * - Auto-suggests from department
    - IT-Software, Sales/BD, Teaching/Training, Operations, HR
- `naukri_industry_type` (VARCHAR 100) * - Auto-suggests
    - EdTech, IT Services, Internet/Ecommerce, SaaS


### **Optional SEO**

- `keywords` (JSONB) - Auto-generates from job title, location, skills
    - ["nodejs developer bangalore", "react jobs india"]

***

# **B. CANDIDATE APPLICATION FORM**

## **SECTION 1: Identity \& Verification** *(30 seconds)*

### **Required Fields**

- `full_name` (VARCHAR 255) *
- `email` (VARCHAR 255) *
- `phone` (VARCHAR 20) * - Country code dropdown
- `otp_code` (VARCHAR 6) * - Email verification
- `alternate_phone` (VARCHAR 20)
- `preferred_contact_method` (VARCHAR 50) - Email, Phone, WhatsApp

***

## **SECTION 2: Professional Summary** *(60 seconds)*

### **Required Fields**

- `current_location` (VARCHAR 255) *
- `total_experience` (VARCHAR 50) * - Dropdown
    - Fresher (0-1 yrs), 1-2 yrs, 2-3 yrs, 3-5 yrs, 5-8 yrs, 8-10 yrs, 10+ yrs
- `resume_file` (FILE) * - PDF/DOC, Max 5MB


### **Conditional: If Experience > 0**

- `current_employment_status` (VARCHAR 50) *
    - Currently Employed, Notice Period, Between Jobs
- `current_company` (VARCHAR 255) *
- `current_designation` (VARCHAR 255) *
- `notice_period_days` (INT) * - input field (in days): 0, 15, 30, 60, 90


### **Optional but Recommended**

- `linkedin_profile` (TEXT)
- `portfolio_website` (TEXT) - For tech/design roles
- `github_profile` (TEXT) - For tech roles
- can add as many as links ( custom website, projects, etc,

***

## **SECTION 3: Education** *(45 seconds)*

### **Required Fields**

- `highest_education` (VARCHAR 100) * ( Degree, Institution, From, End)
    - High School, Diploma, Bachelor's, Master's, PhD
- `institution_name` (VARCHAR 255)
- `cgpa_percentage` (DECIMAL 5,2)
- `degree_name` (VARCHAR 255) * - "B.Tech", "MBA", "BA"
- `specialization` (VARCHAR 255) * - "Computer Science", "Marketing"
- `graduation_year` (INT) *

Allow multiple education to add but optional, but one mandatory

## **SECTION 4: Skills \& Expertise** *(60 seconds)*

- `key_skills` (JSONB) *
    - ["B2B Sales", "CRM Tools", "Client Relations"]
- `languages` (JSONB) *
- `willing_to_relocate` (BOOLEAN)
- `preferred_work_mode` (VARCHAR 50)
    - Remote, Hybrid, On-site, Flexible


## **SECTION 5: Work History** *(90 seconds - for experienced)*

### **Skip for Freshers**

### **For Experienced Candidates**

- `work_experience` (JSONB) - Add current + 1-2 most recent roles

```json
[
  {
    "company": "Tech Corp",
    "designation": "Senior Developer",
    "start_date": "2020-01",
    "end_date": "2024-12",
    "is_current": false,
    "key_achievements": [
      "Led team of 5 developers",
      "Reduced API latency by 40%"
    ]
  }
]
```

    - **Note:** Full resume has complete history, this is just highlights
    
***

## **SECTION 6: Compensation \& Availability** *(30 seconds)*

- `expected_annual_salary` (DECIMAL 12,2) * - In lakhs
    - **Helper:** Show job salary range if disclosed
- `earliest_joining_date` (DATE) *
- `current_annual_salary` (DECIMAL 12,2) - Only if employed

***

## **SECTION 7: Screening Questions** *(30 seconds - if job has questions)*

### **Dynamic Section**

- `screening_answers` (JSONB) - Auto-populated from job settings

```json
[
  {
    "question": "Why do you want to join our company?",
    "answer": "I'm passionate about EdTech..."
  }
]
```


***

## **SECTION 8: Legal Consent** *(Auto-checked)*

### **Required Checkboxes**

- `background_verification_consent` (BOOLEAN) * - Must be true
- `data_privacy_consent` (BOOLEAN) * - GDPR/data protection

***

# **A. JOB POSTING FORM (HR/Recruiter)**

## **SECTION 1: Job Basics**

### **Required Fields**

- `job_code` (VARCHAR 50) - Auto-generated: "DEPT-2025-001"
- `job_title` (VARCHAR 255) *
- `department_id` (INT) * - Reference to departments table
- `job_function` (VARCHAR 100) * - Auto-suggests based on department
- `primary_location` (VARCHAR 255) *
- `alternate_locations` (JSONB) - Renamed plural for clarity
- `work_mode` (VARCHAR 50) *
- `employment_type` (VARCHAR 50) *
- `openings_count` (INT) * - Default: 1
- `contract_duration_months` (INT) - Conditional on employment_type

***

## **SECTION 2: Experience \& Education**

### **Required Fields**

- `experience_level` (VARCHAR 50) *
- `min_experience_years` (DECIMAL 3,1) * - Changed to DECIMAL for "0.5 years" precision
- `max_experience_years` (DECIMAL 3,1) *
- `minimum_education` (VARCHAR 100) *
- `preferred_education` (JSONB) - Optional

***

## **SECTION 3: Skills \& Requirements**

### **Required Fields**

- `required_skills` (JSONB) * - Min 3, Max 10 with auto-suggest[^3][^4]
- `preferred_skills` (JSONB)
- `language_requirements` (JSONB) *

```json
[
  {"language": "English", "level": "Fluent"},
  {"language": "Hindi", "level": "Conversational"}
]
```

- `industry_experience` (JSONB) - For experienced roles only

***

## **SECTION 4: Job Description**

### **Required Fields**

- `job_summary` (TEXT) * - Max 250 chars[^5][^6]
- `responsibilities` (JSONB) * - Min 4, Max 8 bullets (reduced from 12 for readability)

**Optimization Rationale:** Candidates primarily scan responsibilities and summary; additional descriptive fields create redundancy and slow form completion.[^7][^6]

***

## **SECTION 5: Compensation \& Benefits**

### **Salary Configuration**

- `display_salary` (BOOLEAN) * - Default: false
- `salary_type` (VARCHAR 50) * - Conditional
- `min_salary` (DECIMAL 12,2) *
- `max_salary` (DECIMAL 12,2) *
- `currency` (VARCHAR 3) - Default: INR
- `salary_period` (VARCHAR 20) - Default: annual
- `variable_component_percent` (INT) - For sales roles


### **Benefits**

- `benefits` (JSONB) - Multi-select checkboxes[^8]
- `joining_bonus` (DECIMAL 10,2)
- `relocation_support` (BOOLEAN)
***

## **SECTION 6: Application Settings**

### **Required Fields**

- `application_deadline` (DATE)
- `expected_joining_date` (DATE)
- `notice_period_acceptable` (VARCHAR 50) *
- `interview_rounds` (INT) - Default: 3
- `screening_questions` (JSONB) - Max 3 questions recommended[^9][^1]

***

## **SECTION 7: Publishing**

### **Required Fields**

- `publish_immediately` (BOOLEAN) * - Default: true
- `scheduled_publish_at` (TIMESTAMP) - Conditional
- `publish_to_indeed` (BOOLEAN)
- `publish_to_naukri` (BOOLEAN)
- `publish_to_linkedin` (BOOLEAN)

**Naukri Conditionals:**

- `naukri_functional_area` (VARCHAR 100) *
- `naukri_industry_type` (VARCHAR 100) *

***

# **B. CANDIDATE APPLICATION FORM**

## **SECTION 1: Identity \& Verification**

### **Required Fields**

- `full_name` (VARCHAR 255) *
- `email` (VARCHAR 255) *
- `phone` (VARCHAR 20) * - With country code
- `otp_verified` (BOOLEAN) * - Store verification status, not code[^3]
- `alternate_phone` (VARCHAR 20)
- `preferred_contact_method` (VARCHAR 50) - Default: Email

***

## **SECTION 2: Professional Summary**

### **Required Fields**

- `current_location` (VARCHAR 255) *
- `total_experience_years` (DECIMAL 3,1) * - Changed from dropdown to input for precision
- `resume_file` (FILE) * - PDF/DOC, Max 5MB[^7][^3]


### **Conditional: If Experience > 1 year**

- `current_employment_status` (VARCHAR 50) *
- `current_company` (VARCHAR 255) *
- `current_designation` (VARCHAR 255) *
- `notice_period_days` (INT) *


### **Optional**

- `linkedin_profile` (TEXT)
- `portfolio_links` (JSONB) - Combined field for all links (GitHub, portfolio, etc.)[^3]

**Optimization:** One flexible JSONB field instead of separate portfolio_website, github_profile, etc.[^9]

***

## **SECTION 3: Education**

### **Required Fields (One Entry Minimum)**

- `education_history` (JSONB) * - Allow multiple entries[^3]

```json
[
  {
    "degree": "B.Tech",
    "specialization": "Computer Science",
    "institution": "IIT Delhi",
    "graduation_year": 2020,
    "cgpa_percentage": 8.5
  }
]
```

**Optimization:** Store as array rather than separate highest_education fields to avoid redundancy[^11][^1]

***

## **SECTION 4: Skills \& Preferences**

### **Required Fields**

- `key_skills` (JSONB) * - Min 3, Max 15[^4]
- `languages` (JSONB) *
- `willing_to_relocate` (BOOLEAN)
- `preferred_work_mode` (VARCHAR 50)

***

## **SECTION 5: Work History**

### **Skip for Freshers, Conditional for Experienced**

- `work_experience` (JSONB) - Current + 1-2 most recent roles[^3]

```json
[
  {
    "company": "Tech Corp",
    "designation": "Senior Developer",
    "start_date": "2020-01",
    "end_date": "2024-12",
    "is_current": false,
    "key_achievements": ["Led team of 5", "Reduced latency by 40%"]
  }
]
```


***

## **SECTION 6: Compensation \& Availability**

### **Required Fields**

- `expected_annual_salary` (DECIMAL 12,2) *[^6]
- `current_annual_salary` (DECIMAL 12,2) - If employed
- `earliest_joining_date` (DATE) *

***

## **SECTION 7: Screening Questions**

- `screening_answers` (JSONB) - Dynamic from job posting[^1]

***

## **SECTION 8: Legal Consent**

### **Required Checkboxes**

- `background_verification_consent` (BOOLEAN) * - Must be true
- `data_privacy_consent` (BOOLEAN) * - Must be true

***
