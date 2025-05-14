# Developer Guide: Job Application & Scoring System

## Technical Architecture

### Project Structure
```
DjangoProject/
├── jobs/
│   ├── models.py      # Database models
│   ├── views.py       # View functions
│   ├── forms.py       # Form classes
│   ├── urls.py        # URL patterns
│   ├── templates/     # HTML templates
│   │   ├── apply_to_job.html
│   │   ├── view_application.html
│   │   └── view_applicants.html
│   ├── static/        # CSS, JS, images
│   ├── signals.py     # Model signals
│   └── tests.py       # Unit tests
└── DjangoProject/     # Project directory
    ├── settings.py
    ├── urls.py
    └── wsgi.py
```
The `jobs/utils/` directory containing `scoring.py` and `fuzzy_matching.py` would be removed as scoring is no longer part of the system.

## Core Models

### Job Model
```python
class Job(models.Model):
    title = models.CharField(max_length=100)
    # ... other fields like description, location, requirements ...
    posted_by = models.ForeignKey(User, on_delete=models.CASCADE)
    # Fields for job requirements (e.g., education_required, experience_required)
    education_required = models.TextField(blank=True, null=True)
    experience_required = models.TextField(blank=True, null=True)
    training_required = models.TextField(blank=True, null=True)
    eligibility_required = models.TextField(blank=True, null=True)
    # ...
```

### ResumeApplication Model
```python
class ResumeApplication(models.Model):
    job = models.ForeignKey(Job, on_delete=models.CASCADE, related_name='applications')
    name = models.CharField(max_length=100)
    email = models.EmailField()
    phone = models.CharField(max_length=20)
    title = models.CharField(max_length=100)  # Professional title
    location = models.CharField(max_length=100, null=True, blank=True)
    objective = models.TextField(null=True, blank=True)
    
    # JSON Fields for structured data
    experiences = models.JSONField(null=True, blank=True)
    education = models.JSONField(null=True, blank=True)
    skills = models.JSONField(null=True, blank=True) # Retained for general skill listing
    trainings = models.JSONField(null=True, blank=True) # formerly seminars
    elisgibility = models.JSONField(null=True, blank=True) # New field
    references = models.JSONField(null=True, blank=True)
    
    submitted_at = models.DateTimeField(auto_now_add=True)
    
    def __str__(self):
        return f"{self.name} - {self.job.title}"
        
    # calculate_score and compute_score methods are removed.
```
The `Score` model is removed.

## Key Views Implementation

### Apply to Job View
```python
# In jobs/views.py
def apply_to_job(request, job_id):
    """Handle job application submission and processing"""
    job = get_object_or_404(Job, id=job_id) # Simplified, assuming is_active is handled elsewhere or not needed
    
    if request.method == 'POST':
        # Process form data into structured format
        # This part remains largely the same, extracting data from request.POST
        experiences = process_experience_data(request.POST) # Assuming these functions exist
        education = process_education_data(request.POST)
        skills = process_skills_data(request.POST)
        trainings = process_trainings_data(request.POST) # formerly process_seminars_data
        eligibility = process_eligibility_data(request.POST) # New
        references = process_references_data(request.POST)
        
        # Create application
        application = ResumeApplication(
            job=job,
            name=request.POST.get('name'),
            email=request.POST.get('email'),
            phone=request.POST.get('phone'),
            title=request.POST.get('title'),
            location=request.POST.get('location'),
            objective=request.POST.get('objective'),
            experiences=experiences,
            education=education,
            skills=skills, # skills are still collected
            trainings=trainings, # formerly seminars
            elisgibility=eligibility, # new field
            references=references,
        )
        application.save()
        
        # Score calculation is removed.
        
        # Redirect to success page
        messages.success(request, "Application submitted successfully!")
        return redirect('job_detail', job_id=job.id) # Or a relevant success page
    
    # GET request - show form
    return render(request, 'apply_to_job.html', {'job': job})
```

### View Application View
```python
# In jobs/views.py
@login_required
def view_application(request, application_id):
    application = get_object_or_404(ResumeApplication, id=application_id)
    # Permission checks remain
    # ...
    context = {
        'application': application,
        'job': application.job,
        # Other necessary fields from application model for display
        # Score related context is removed.
    }
    return render(request, 'view_application.html', context)
```

### View Applicants View
```python
# In jobs/views.py
@login_required
def view_applicants(request, job_id):
    job = get_object_or_404(Job, id=job_id)
    # Permission checks remain
    # ...
    applicants = ResumeApplication.objects.filter(job=job).order_by('-submitted_at') # Or other relevant ordering
    # Score-based sorting is removed.
    context = {
        'job': job,
        'applicants': applicants,
    }
    return render(request, 'view_applicants.html', context)
```

## Data Processing Functions
Helper functions (`process_experience_data`, `process_education_data`, etc.) in `views.py` or a utility file would still be needed to structure data from the POST request for the JSON fields in `ResumeApplication`. These are not directly related to scoring.

## Extending the System

### Adding New Fields to the Application
To add new information to the `ResumeApplication`:
1.  Update the `ResumeApplication` model in `models.py` with the new field(s).
2.  Update the application form template (`apply_to_job.html`) to include input(s) for the new field(s).
3.  Update the `apply_to_job` view in `views.py` to process the new field(s) from `request.POST` and save them to the `ResumeApplication` instance.
4.  Update the `view_application.html` template to display the new field(s).

## Testing
Unit tests in `tests.py` would need to be updated:
*   Remove tests related to score calculation (`test_score_calculation`).
*   Tests for application submission and viewing should be adjusted to not expect score-related data or behavior.
*   Focus tests on the correct saving and retrieval of application data.

Example of an updated test:
```python
# in tests.py
from django.test import TestCase
from django.urls import reverse
from .models import Job, ResumeApplication, User # User for posted_by

class ApplicationViewsTests(TestCase):
    def setUp(self):
        self.test_user = User.objects.create_user('testemployer', 'test@example.com', 'password')
        self.job = Job.objects.create(
            title='Test Developer Position',
            description='Test description',
            # requirements='Test requirements', # Assuming requirements is not a direct field anymore
            posted_by=self.test_user
        )
        # Minimal data for application creation, as scoring is removed
        self.application_data = {
            'name': 'Test Applicant',
            'email': 'applicant@example.com',
            'phone': '1234567890',
            # Add other required fields for ResumeApplication model
        }

    def test_apply_to_job_submission(self):
        """Test that an application can be submitted."""
        response = self.client.post(reverse('apply_to_job', args=[self.job.id]), self.application_data)
        self.assertEqual(response.status_code, 302) # Should redirect on success
        self.assertTrue(ResumeApplication.objects.filter(job=self.job, name='Test Applicant').exists())

    def test_view_application_details(self):
        """Test viewing a submitted application."""
        # Create an application first
        app = ResumeApplication.objects.create(
            job=self.job, 
            name='View Applicant', 
            email='view@example.com', 
            phone='0987654321'
            # ... other fields
        )
        self.client.login(username='testemployer', password='password') # Login as employer
        response = self.client.get(reverse('view_application', args=[app.id]))
        self.assertEqual(response.status_code, 200)
        self.assertContains(response, 'View Applicant')
        # self.assertNotContains(response, 'Score') # Ensure score is not displayed

    # Other tests for view_applicants, etc.
```

## Deployment Notes

### Dependencies
- Django 4.1+
- Postgres (recommended for production)
- Python 3.8+
(Scoring related dependencies like fuzzywuzzy would be removed)

### Configuration
Key settings to configure:
- Database connection details in `settings.py`
- Email configuration for notifications
- Static file serving (use CDN in production)
- Session and cache settings for performance

### Performance Considerations
For large-scale deployments:
- Implement database query optimizations and indexes, especially for `ResumeApplication` filtering.
- Cache frequently accessed data (job listings, aggregate stats if any).
- Use pagination for applicant listings with many entries.

## Security Considerations

The application implements several security measures:

1.  **Input Validation**
    *   Form validation for all input fields.
    *   Sanitization of user-provided content (Django templates auto-escape by default).
2.  **Access Control**
    *   Role-based permissions (employer vs. applicant) using Django's auth system and custom Profile model.
    *   Object-level permissions (employers can only see applications for their own jobs).
3.  **Data Protection**
    *   CSRF protection on all forms (Django default).
    *   SQL injection prevention through ORM (Django default).
    *   Consider encryption for sensitive PII if stored beyond standard Django user model fields, though JSONFields are stored as text/jsonb in DB.
4.  **Audit Trail**
    *   Timestamps on all application actions (`submitted_at`).
    *   Django Admin logs actions. Custom logging for critical application events can be added.