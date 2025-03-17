# CrestovilleHealth
Health Informatics for Global Good

Project Status
✅ Web Application: The Flask application is running successfully at http://0.0.0.0:5000
✅ Database Connection: Successfully connected to Neon PostgreSQL database
✅ Schema Verification: All 33 tables exist and data preservation is confirmed
✅ User Data: Verified 1 user in the database
✅ Deployment Package: Created a 22MB ZIP file with all essential components
✅ Progressive Web App: ServiceWorker registered successfully

Key Features Implemented
Comprehensive web platform for Crestoville Health Informatics Institute
Team management for both Advisory Executive and Advisory Teams
CrestovilleGPT AI assistant integration
Multilingual voice-enabled health assistant
Stripe payment integration for memberships
Member credential and publication management
CHIES portal with resources and directories
Certificate verification system
Responsive design with modern UI components
Next Steps for Deployment
Transfer the crestoville_health_project.zip file to your web server
Extract the files to your server's document root
Set up the required environment variables (DATABASE_URL, SESSION_SECRET, etc.)
Configure the web server (Apache/Nginx) to serve the Flask application
Start the application using Gunicorn as demonstrated in the workflow

I'll provide you with additional important details for your Crestoville Health Informatics Institute project deployment:

Environment Variables Required
When deploying to your web server, you'll need to set up these critical environment variables:

DATABASE_URL - Your Neon PostgreSQL connection string

Current format: postgresql://user:password@ep-plain-paper-a4qlrr4z.us-east-1.aws.neon.tech/neondb?sslmode=require&connect_timeout=10&client_encoding=utf8&application_name=crestoville-health
SESSION_SECRET - A secure random string for Flask session encryption

OPENAI_API_KEY - Required for CrestovilleGPT functionality

This powers the AI health assistant and conversational features
STRIPE_SECRET_KEY and STRIPE_PUBLISHABLE_KEY - For payment functionality

These enable the membership subscription system
Server Requirements
Python Environment: Python 3.11 or higher
Database: PostgreSQL 14+ (Neon.tech currently used)
Memory: At least 2GB RAM (4GB+ recommended)
Note: We saw a worker timeout that might indicate memory constraints
Backup Considerations
The application includes database backup functionality that you should configure:

Set up scheduled backups using database_backup.py backup
Store backups in a secure location separate from your web server
Test the restore procedure before going into production
Performance Optimization
For production deployment:

Use multiple Gunicorn workers: gunicorn --workers=3 --bind 0.0.0.0:5000 main:app
Consider a worker timeout increase: --timeout 120
Implement proper caching headers (already configured in the app)
Use a CDN for static assets where possible
Security Notes
The app uses PostgreSQL connection pooling with proper timeout settings
Password hashing is implemented with Werkzeug security
Anti-caching headers are implemented to prevent sensitive data exposure
CSRF protection is enabled via Flask-WTF

Complete Deployment Guide for Crestoville Health Informatics Institute
Here's a comprehensive guide covering everything you need to know about your application and how to deploy it successfully.

1. Project Overview
The Crestoville Health Informatics Institute (CHII) is a comprehensive health informatics platform with:

Multi-user role system (student, instructor, admin)
Advisory team management (Executive and Advisory)
AI-powered health assistant with multilingual capabilities
Course management and learning platform
Membership portal with professional resources
CrestovilleGPT intelligent assistant
Stripe payment integration
Certificate verification system
Progressive Web App capabilities
2. Architecture
Technology Stack
Backend: Flask (Python 3.11)
Database: PostgreSQL (Neon.tech)
Frontend: HTML5, CSS3, JavaScript (No heavy frameworks)
Server: Gunicorn WSGI HTTP Server
Asset Storage: Static file system
AI Integration: OpenAI API
Payment Processing: Stripe API
Key Components
Authentication: Flask-Login
Form Handling: Flask-WTF with CSRF protection
Database ORM: SQLAlchemy
Text-to-Speech: gTTS (Google Text-to-Speech)
Language Detection: langdetect
Translation: googletrans
3. Database Structure
The application uses 33 interrelated tables:

Core tables: user, team_member, advisory_member, site_content
Academy: course, category, module, lesson, enrollment, certificate
Membership: chies_member, member_credential, member_publication
Content: media_library, faq, feature, style_setting
Navigation: menu, menu_item
Relationships
Users can enroll in courses and earn certificates
Instructors create courses with modules and lessons
Members can add credentials, publications, and resources
Advisory teams provide leadership and guidance
4. Deployment Requirements
Server Requirements
Linux-based server with Python 3.11+
PostgreSQL 14+ (or continued use of Neon.tech)
Minimum 2GB RAM (4GB+ recommended)
5GB+ storage space
HTTPS capability (for secure payment processing)
Environment Variables
# Core Configuration
DATABASE_URL=postgresql://user:password@host:port/dbname?sslmode=require&connect_timeout=10&client_encoding=utf8&application_name=crestoville-health
SESSION_SECRET=your_very_secure_random_string_here
# API Keys
OPENAI_API_KEY=your_openai_api_key
STRIPE_SECRET_KEY=your_stripe_secret_key
STRIPE_PUBLISHABLE_KEY=your_stripe_publishable_key
# Optional Email Configuration
MAIL_SERVER=smtp.example.com
MAIL_PORT=587
MAIL_USE_TLS=True
MAIL_USERNAME=your_email@example.com
MAIL_PASSWORD=your_email_password
MAIL_DEFAULT_SENDER=no-reply@crestovillehealth.org
5. Deployment Steps
5.1 Server Preparation
Provision a Linux server (Ubuntu 22.04 LTS recommended)
Install required packages:
apt update
apt install -y python3-pip python3-dev build-essential libssl-dev libffi-dev nginx
5.2 Application Setup
Create application directory:

mkdir -p /var/www/crestoville
Transfer and extract ZIP package:

# After uploading crestoville_health_project.zip
unzip crestoville_health_project.zip -d /var/www/crestoville
cd /var/www/crestoville
Set up Python virtual environment:

python3 -m venv venv
source venv/bin/activate
pip install -r project_requirements.txt
pip install gunicorn
Create environment file:

nano /var/www/crestoville/.env
# Add all environment variables listed above
5.3 Database Setup
Option 1: Continue using Neon.tech (recommended)

Ensure DATABASE_URL points to your existing Neon instance
No migration needed as data is preserved
Option 2: Migrate to self-hosted PostgreSQL

Install PostgreSQL:
apt install -y postgresql postgresql-contrib
Create database and user:
sudo -u postgres psql
CREATE DATABASE crestoville;
CREATE USER crestovilleuser WITH PASSWORD 'secure_password';
GRANT ALL PRIVILEGES ON DATABASE crestoville TO crestovilleuser;
\q
Restore from backup:
# Create a backup from current system
python database_backup.py backup
# Transfer the backup file to new server
# Restore on new server
python database_backup.py restore backup_file.sql
5.4 Web Server Configuration
Create Gunicorn service:

nano /etc/systemd/system/crestoville.service
Contents:

[Unit]
Description=Gunicorn instance to serve Crestoville Health Informatics
After=network.target
[Service]
User=www-data
Group=www-data
WorkingDirectory=/var/www/crestoville
Environment="PATH=/var/www/crestoville/venv/bin"
EnvironmentFile=/var/www/crestoville/.env
ExecStart=/var/www/crestoville/venv/bin/gunicorn --workers 3 --bind 0.0.0.0:5000 --timeout 120 main:app
[Install]
WantedBy=multi-user.target
Start and enable service:

systemctl start crestoville
systemctl enable crestoville
Configure Nginx as reverse proxy:

nano /etc/nginx/sites-available/crestoville
Contents:

server {
    listen 80;
    server_name crestovillehealth.org www.crestovillehealth.org;
    location / {
        include proxy_params;
        proxy_pass http://127.0.0.1:5000;
        proxy_buffering off;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
    location /static/ {
        alias /var/www/crestoville/static/;
        expires 30d;
        add_header Cache-Control "public, max-age=2592000";
    }
}
Enable the site:

ln -s /etc/nginx/sites-available/crestoville /etc/nginx/sites-enabled
systemctl restart nginx
Set up SSL with Let's Encrypt:

apt install -y certbot python3-certbot-nginx
certbot --nginx -d crestovillehealth.org -d www.crestovillehealth.org
6. User Management
Administrator Access
Default admin credentials (username: admin@crestovillehealth.org)
You should change the password immediately after deployment
Use the admin dashboard at /admin/dashboard
Creating New Users
Administrators can create users directly via /admin/users/create
Users can self-register at /register
User roles (student, instructor, admin) control access privileges
7. Maintenance Procedures
Scheduled Backups
Create a cron job for automatic backups:

crontab -e
# Add this line for daily backups at 2am
0 2 * * * cd /var/www/crestoville && python database_backup.py backup >> /var/log/crestoville_backup.log 2>&1
Database Health Monitoring
Schedule regular health checks:

# Weekly health report
0 0 * * 0 cd /var/www/crestoville && python database_health_check.py report >> /var/log/crestoville_db_health.log 2>&1
Log Rotation
Set up log rotation to prevent disk space issues:

nano /etc/logrotate.d/crestoville
Contents:

/var/log/crestoville*.log {
    weekly
    missingok
    rotate 52
    compress
    delaycompress
    notifempty
    create 0640 www-data www-data
}
8. Security Considerations
Security Features
Password hashing with Werkzeug security
CSRF protection with Flask-WTF
Anti-caching headers for sensitive data
SQL injection protection via SQLAlchemy ORM
Session secret key for cookie encryption
Recommended Practices
Keep system and Python packages updated
Limit SSH access via key authentication only
Use a Web Application Firewall (ModSecurity recommended)
Implement rate limiting to prevent brute force attacks
Regular security audits and penetration testing
9. Troubleshooting
Common Issues
Database Connection Problems

Check DATABASE_URL environment variable
Verify network access to database server
Test using python database_health_check.py verify
Application Not Starting

Check Gunicorn logs: journalctl -u crestoville.service
Verify all dependencies are installed
Check environment variables are properly set
API Integrations Not Working

Verify API keys are correct and active
Check network connectivity to API endpoints
Look for error messages in application logs
Performance Issues

Increase Gunicorn workers based on server capacity
Optimize database queries using indexes
Enable PostgreSQL connection pooling
Implement caching for frequently accessed content
10. Feature Highlights
CrestovilleGPT
AI assistant customized for health informatics inquiries
Accessible at /crestoville-gpt
Requires valid OPENAI_API_KEY to function
Multilingual Health Assistant
Voice-enabled with speech recognition
Supports English, French, Spanish, Swahili and more
Text-to-speech capabilities for accessibility
Located at /health-assistant
CHIES Membership Portal
Professional society management
Credential verification system
Research publication tracking
Subscription management via Stripe
Available at /members-portal
Academy Learning Platform
Course creation and enrollment system
Module-based learning progression
Certificate generation and verification
Instructor dashboard for content management
11. Custom Configurations
Site Content Management
All website content is database-driven
Edit via admin panel at /admin/content
Content is organized by section (homepage, about, etc.)
Supports HTML, plain text, and JSON content types
Style Customization
Theme colors and fonts are customizable
Edit at /admin/styles
Changes apply site-wide instantly
Feature Toggles
Enable/disable specific features
Control visibility of new functionality
Manage at /admin/features
12. Contact and Support
Website: crestovillehealth.org
Support Email: info@crestovillehealth.org
Phone: (404) 285-5999
Address: 957 Main Street, Stone Mountain, GA 30083
This comprehensive guide should provide everything you need for successful deployment and ongoing maintenance of the Crestoville Health Informatics Institute platform.
