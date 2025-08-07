# Technical Requirements: Employee Reporting System (ERS)



## **1. Core Concepts \& User Roles**

The system will be built around a clear hierarchy and defined roles.

* **1.1. User Roles**
    * **Employee:** The base user. Can create, submit, and manage their own reports. They can view their personal reporting history and statistics.
    * **Manager:** An Employee who also has direct reports. Inherits all Employee permissions and gains additional capabilities to view, approve, and analyze the reports of their team members. The system must support multi-level hierarchies (a manager reporting to another manager).

## **2. Report Management Module**

This is the core module for creating, tracking, and viewing reports.

* **2.1. Report Creation \& Submission (Employee)**
    * **Report Types:** The Admin can define various report templates (e.g., Daily Reports, Weekly Summary, Monthly Review, Project Update).
    * **Rich Text Editor:** Implement a rich text editor like `Tiptap`-based editor supporting:
        * Standard formatting (headings, bold, italics, lists, links, alignments, colors, text formatting features).
        * Code blocks and blockquotes, links.
        * Images and table 
        * **@Mentions (optional):** Tagging team members within a report, which triggers an in-app notification for the tagged user.
    * **Project Tagging (Optional):** An optional feature to associate a report with one or more predefined projects.
    * **Attachments:**
        * Users can upload up to 5 files per report (configurable by Admin).
        * Files will be stored securely as private assets via a service like Cloudinary.
    * **Drafts:**
        * Reports are auto-saved as drafts every 60 seconds.
        * Users can manually save, view, edit, or delete drafts.
        * A draft for a specific reporting period (e.g., today's daily report) cannot be submitted after the deadline has passed without manager approval.
* **2.2. Report Lifecycle \& Status**
A report will exist in one of the following states:
    * `Draft`: Saved but not yet submitted.
    * `Submitted`: Manager can see the report.
    * `Pending Edit Approval`: An employee has requested to edit a submitted report after the deadline.
    * `Approved / Rejected`: A manager's decision on an edit request.
    * `Missed`: Automatically logged by the system if no report is submitted for a required workday. This is crucial for accurate consistency tracking.
* **2.3. Post-Submission Edits \& Deletion**
    * **Before Deadline:** Employees can freely edit or delete their own submitted reports for the current reporting period.
    * **After Deadline (Approval Workflow):**

	1. Employee requests to edit a submitted report, providing a reason.
	2. Manager receives a notification and can view the request, also see the request in reports dashboard with report details and notes.
	3. Manager can `Approve` or `Reject` the request.
	4. If approved, the report becomes editable for the employee to resubmit (only once or, throught the day).


#### **3. Dashboards \& Viewing**

* **3.1. My Reports Dashboard (Employee)**
    * A personalized view showing:
        * A list of all submitted, draft, and missed reports with status indicators.
        * A calendar view to visualize reporting consistency at a glance. (github like style)
        * Personal reporting statistics (e.g., submission percentage for the month).
* **3.2. Team Reports Dashboard (Manager)**
    * **Team Overview:** High-level analytics for the team, including overall submission rate, consistency leaders, and frequent defaulters.
    * **Report Feed:** A consolidated view of all team members' reports, sortable and groupable by employee or date.
    * **Efficient Loading:** Backend pagination is essential to handle large volumes of reports without slowing down the UI.
    * **Direct Actions:** Managers can view full report details (content, tags, attachments) and approve/reject edit requests directly from this dashboard.
* **3.3. Advanced Search \& Filtering**
    * **Global Filters:** Filter all report views by:
        * Date or Date Range
        * Employee(s) or Department(s)
        * Report Status (`Submitted`, `Missed`, etc.)
        * Report Type (`Daily`, `Weekly`, etc.)
        * Project Tag
    * **Full-Text Search:** A powerful search engine (e.g., Elasticsearch) to query the content of reports, comments, and mentions.


#### **4. Analytics \& AI Module**

* **4.1. Performance \& Consistency Analytics**
    * **Consistency Score:** Calculated for each employee as: `(Submitted Reports / (Total Workdays - Approved Leave Days)) * 100%`.
    * **Performance Flags:** The system automatically flags:
        * **Defaulters:** Employees with consistency below a configurable threshold (e.g., 80%).
        * **Top Performers:** Employees with consistently high submission rates.
    * **Trend Visualization:** Charts showing reporting trends over time for individuals, teams, or the entire organization.
* **4.2. AI-Powered Features (Optional Add-on)**
    * **AI Rewrite Assist:** An in-editor button for employees to "Improve Writing" or "Check Grammar \& Tone" before submission.
    * **AI Report Summarizer:** On-demand or scheduled generation of summaries for a manager's team.
        * **Scope:** Weekly, monthly, or custom date range summaries.
        * **Content:** The AI will identify key achievements, recurring blockers, project progress, and overall sentiment from the reports.


#### **5. System Administration \& Configuration (Admin Role)**

* **5.1. User \& Department Management:**
    * Create, update, deactivate users.
    * Assign roles (Employee, Manager, Admin).
    * Define the organizational hierarchy (who reports to whom).
* **5.2. Holiday \& Leave Management:**
    * **Holiday Calendars:** Admins can create and manage multiple holiday lists (e.g., by country or region).
    * **Leave Integration:** A simple system for employees to mark themselves on leave (Vacation, Sick Day). This is critical for excluding these days from consistency calculations.
* **5.3. System Settings:**
    * Configure the daily report submission deadline (e.g., 5:00 PM local time).
    * Manage report types, project tags, and notification preferences.
    * Set the data retention and archiving policies.


#### **6. Notifications \& Reminders**

Notifications will be delivered both in-app and via email.

* **Submission Reminders:** Daily reminder to all employees with a pending report, sent 1 hour before the deadline.
* **Activity Alerts (for Managers):** Real-time notifications when a team member submits a report or requests an edit.
* **AI Summary Alerts:** Notification when a requested or scheduled AI summary is ready for viewing.
* **Weekly Digest:** An automated email sent to managers every Friday EOD with a summary of their team's reporting stats for the week.


#### **7. Data Export \& Archiving**

* **Exporting:**
    * Export a single report as a clean, well-formatted PDF.
    * Bulk export options (e.g., "Export all my reports for July").
    * Option to include attachments as a zipped file in the export.

