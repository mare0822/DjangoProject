# Job Application and Scoring System Documentation

## Table of Contents
1. [Overview](#overview)
2. [System Architecture](#system-architecture)
3. [User Roles](#user-roles)
4. [Application Process](#application-process)
5. [Resume Structure](#resume-structure)
<!-- Removed Scoring System -->
6. [Views and Templates](#views-and-templates)
7. [Database Models](#database-models)
8. [Implementation Guide](#implementation-guide)
<!-- Removed Score Interpretation -->
9. [Conclusion](#conclusion)


## Overview

The Job Application System is a Django-based web application that allows job seekers to apply for positions and employers to review these applications. The system stores detailed resume information submitted by applicants.
<!-- Removed "and calculates scores..." and AEA framework mention -->

## System Architecture

The application follows a standard Django MVT (Model-View-Template) architecture:

1.  **Models** - Store job listings, user profiles, and applications. <!-- Removed scoring data -->
2.  **Views** - Handle user interactions, form submissions, and page rendering.
3.  **Templates** - Render UI components for users.
4.  **URLs** - Map routes to appropriate views.

```
DjangoProject/
├── jobs/
│   ├── models.py      # Database models
│   ├── views.py       # View functions
│   ├── urls.py        # URL mappings
│   ├── templates/     # HTML templates
│   ├── static/        # CSS, JS, images
│   └── signals.py     # Model signals
├── manage.py          # Django command-line utility
└── DjangoProject/     # Project settings
    └── settings.py    # Project configuration
```
<!-- Removed jobs/utils/scoring.py and fuzzy_matching.py if they were listed -->

## User Roles

The system supports two main user roles:

1.  **Job Seekers**
    *   Register and create profiles.
    *   Browse available job listings.
    *   Submit detailed resume applications.
    *   Track application status (basic).
2.  **Employers**
    *   Post job listings with requirements.
    *   View submitted applications.
    <!-- Removed "Access detailed scoring for each applicant" and "Compare applicants via scoring metrics" -->

## Application Process

The job application process follows these steps:

1.  **Job Posting**: Employer creates a job listing with title, description, location, and qualifications/requirements.
2.  **Job Discovery**: Job seekers browse or search for jobs.
3.  **Application**: Job seeker completes the application form with detailed resume information.
4.  **Submission**: Application is stored. <!-- Removed "and immediately scored..." -->
5.  **Review**: Employer receives notification of new application and can view the resume. <!-- Removed "and score" -->
6.  **Decision**: Employer uses resume information to make hiring decisions. <!-- Removed "score and" -->

## Resume Structure

The resume application form captures comprehensive candidate information:

1.  **Personal Information**
    *   Full name, professional title, contact info.
    *   Career objective.
2.  **Experience**
    *   Job titles and companies.
    *   Employment dates and locations.
    *   Detailed job descriptions.
3.  **Education**
    *   Institutions and locations.
    *   Degrees/courses and graduation years.
4.  **Skills**
    *   Technical and soft skills.
5.  **Trainings & Seminars** (formerly Seminars & Training)
    *   Professional development activities.
    *   Organizers and dates.
6.  **Eligibility/Certifications** (New Section)
    *   Details of any professional licenses, certifications, or eligibilities.
7.  **References**
    *   Professional references and contact information.

<!-- Removed Scoring System section -->

## Views and Templates

### Key Templates

1.  **`apply_to_job.html`**
    *   Dynamic form for resume submission.
    *   Multiple sections for different resume components.
    *   JavaScript for adding additional entries (e.g., for multiple experiences).
2.  **`view_application.html`**
    *   Detailed resume view for employers.
    *   Summary of applicant information.
    <!-- Removed "Link to score details" -->
3.  **`view_applicants.html`**
    *   Table of all applicants for a job.
    <!-- Removed "Summary scores with color coding" -->
    *   Links to detailed application views.
<!-- Removed `score_detail.html` -->

### Key Views

1.  **`apply_to_job(request, job_id)`**
    *   Handles resume form submission.
    *   Creates `ResumeApplication` record.
    <!-- Removed "Triggers score calculation" -->
2.  **`view_application(request, application_id)`**
    *   Displays detailed resume for employers.
    *   Formats data for template rendering.
3.  **`view_applicants(request, job_id)`**
    *   Shows all applicants for a specific job.
    <!-- Removed "Sorts by score for easy comparison" -->
<!-- Removed `view_score_details(request, application_id)` and its description -->

## Database Models

### Core Models

1.  **User & Profile**
    *   Django's built-in `User` model.
    *   Extended `Profile` with role (employer/seeker).
2.  **Job**
    *   Basic job listing information.
    *   Posted by (employer reference).
    *   Location, title, description, requirements fields.
3.  **ResumeApplication** (formerly Application or combined with it)
    *   Detailed resume information.
    *   JSON fields for structured data (experiences, education, skills, trainings, eligibility, references).
    *   Foreign key to `Job`.
<!-- Removed `Score` model -->

## Implementation Guide

### Setting Up a New Job

1.  Log in as an employer.
2.  Navigate to employer dashboard.
3.  Click "Create New Job".
4.  Enter job title, description, location, and specific requirements (education, experience, training, eligibility).
5.  Submit to publish the job.

### Applying for a Job

1.  Log in as a job seeker (or apply as guest if allowed).
2.  Browse available jobs.
3.  Select a job and click "Apply".
4.  Fill out the detailed resume form.
5.  Submit application.

### Viewing Applications (for Employers)

1.  Log in as an employer.
2.  Navigate to employer dashboard.
3.  Select a job to view applicants.
4.  View the list of applicants. <!-- Removed "with their scores" -->
5.  Click on individual applicants to view detailed resumes.
<!-- Removed "Click 'View Score' to see detailed scoring breakdown" -->

<!-- Removed "Recalculating Scores" section -->
<!-- Removed "Score Interpretation" section -->

## Conclusion

This documentation provides a comprehensive overview of the Job Application System. For further technical details, refer to the code comments within each file. The system is designed to be extensible, allowing for additional features to be added as needed.