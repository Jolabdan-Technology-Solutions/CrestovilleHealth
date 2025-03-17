Below is a step-by-step guide to deploy the Crestoville Health Informatics Institute (CHII) Flask web application onto your Linode server, rewritten from the provided documentation. This guide assumes you have a Linode server running Ubuntu 22.04 LTS and covers transferring the application, setting up the environment, configuring the server, and linking it to a custom domain. Code snippets and commands are included for clarity.
Step-by-Step Guide to Deploy CHII Flask App on Linode
Step 1: Prepare Your Linode Server
Objective: Set up a Linode server with the necessary dependencies.
Provision a Linode Instance:
Log into your Linode account and create a new Linode.

Choose Ubuntu 22.04 LTS as the distribution.

Select a plan (e.g., 2GB RAM Nanode minimum, 4GB+ recommended).

Deploy the server and note its public IP address (e.g., 192.168.1.1).

SSH into the Server:
bash

ssh root@192.168.1.1

Update and Install Base Packages:
bash

apt update && apt upgrade -y
apt install -y python3-pip python3-dev build-essential libssl-dev libffi-dev nginx postgresql postgresql-contrib

Create a Non-Root User (for security):
bash

adduser --gecos "" crestoville
usermod -aG sudo crestoville
su - crestoville

Step 2: Transfer and Set Up the Application
Objective: Upload the crestoville_health_project.zip file and configure the app environment.
Create Application Directory:
bash

sudo mkdir -p /var/www/crestoville
sudo chown crestoville:crestoville /var/www/crestoville
cd /var/www/crestoville

Transfer the ZIP File:
From your local machine, use SCP to upload the 22MB ZIP file:
bash

scp crestoville_health_project.zip crestoville@192.168.1.1:/var/www/crestoville/

On the server:
bash

unzip crestoville_health_project.zip

Set Up Python Virtual Environment:
bash

python3 -m venv venv
source venv/bin/activate
pip install -r project_requirements.txt
pip install gunicorn

Configure Environment Variables:
Create an .env file:
bash

nano .env

Add the following (replace placeholders with your actual values):
bash

DATABASE_URL=postgresql://user:password@ep-plain-paper-a4qlrr4z.us-east-1.aws.neon.tech/neondb?sslmode=require&connect_timeout=10&client_encoding=utf8&application_name=crestoville-health
SESSION_SECRET=your_very_secure_random_string_here
OPENAI_API_KEY=your_openai_api_key
STRIPE_SECRET_KEY=your_stripe_secret_key
STRIPE_PUBLISHABLE_KEY=your_stripe_publishable_key
SITE_URL=https://crestovillehealth.org

Save and exit (Ctrl+O, Enter, Ctrl+X).

Step 3: Database Setup
Objective: Use the existing Neon PostgreSQL database or migrate to a self-hosted instance.
Option 1: Continue Using Neon.tech (Recommended):
Verify the DATABASE_URL in .env points to your Neon instance.

Test connectivity:
bash

source venv/bin/activate
python -c "import os; from sqlalchemy import create_engine; engine = create_engine(os.getenv('DATABASE_URL')); conn = engine.connect(); print('Connected!'); conn.close()"

If "Connected!" prints, no further action is needed.

Option 2: Migrate to Self-Hosted PostgreSQL (if preferred):
Create a database and user:
bash

sudo -u postgres psql
CREATE DATABASE crestoville;
CREATE USER crestovilleuser WITH PASSWORD 'secure_password';
GRANT ALL PRIVILEGES ON DATABASE crestoville TO crestovilleuser;
\q

Update .env with new DATABASE_URL:
bash

DATABASE_URL=postgresql://crestovilleuser:secure_password@localhost:5432/crestoville

Restore from backup:
Generate a backup from Neon locally:
bash

python database_backup.py backup

Transfer the backup file (e.g., backup.sql) to Linode:
bash

scp backup.sql crestoville@192.168.1.1:/var/www/crestoville/

Restore on Linode:
bash

psql -U crestovilleuser -d crestoville -f backup.sql

Step 4: Configure Gunicorn Service
Objective: Set up Gunicorn to run the Flask app as a systemd service.
Create Service File:
bash

sudo nano /etc/systemd/system/crestoville.service

Add:
ini

[Unit]
Description=Gunicorn instance for Crestoville Health Informatics
After=network.target

[Service]
User=crestoville
Group=www-data
WorkingDirectory=/var/www/crestoville
Environment="PATH=/var/www/crestoville/venv/bin"
EnvironmentFile=/var/www/crestoville/.env
ExecStart=/var/www/crestoville/venv/bin/gunicorn --workers 3 --bind 0.0.0.0:5000 --timeout 120 main:app

[Install]
WantedBy=multi-user.target

Save and exit.

Start and Enable Service:
bash

sudo systemctl start crestoville
sudo systemctl enable crestoville

Verify Status:
bash

sudo systemctl status crestoville

Look for "active (running)" in the output.

Step 5: Configure Nginx as Reverse Proxy
Objective: Use Nginx to serve the app publicly and handle static files.
Create Nginx Config:
bash

sudo nano /etc/nginx/sites-available/crestoville

Add:
nginx

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

Save and exit.

Enable the Site:
bash

sudo ln -s /etc/nginx/sites-available/crestoville /etc/nginx/sites-enabled
sudo nginx -t  # Test configuration
sudo systemctl restart nginx

Step 6: Set Up SSL with Let’s Encrypt
Objective: Secure the app with HTTPS.
Install Certbot:
bash

sudo apt install -y certbot python3-certbot-nginx

Generate SSL Certificate:
bash

sudo certbot --nginx -d crestovillehealth.org -d www.crestovillehealth.org

Follow prompts: provide an email, agree to terms, and choose to redirect HTTP to HTTPS.

Verify HTTPS:
Visit https://crestovillehealth.org in a browser; ensure the lock icon appears.

Step 7: Link to Custom Domain
Objective: Point your domain (e.g., crestovillehealth.org) to the Linode server.
DNS Configuration:
Log into your domain registrar (e.g., Namecheap, GoDaddy).

Add DNS records:
A Record (root domain):

Type: A
Host: @
Value: 192.168.1.1  # Your Linode IP
TTL: 3600

CNAME Record (www subdomain):

Type: CNAME
Host: www
Value: crestovillehealth.org
TTL: 3600

Wait for DNS Propagation:
Use dig crestovillehealth.org or a tool like What’s My DNS to confirm the IP updates (may take 24-48 hours).

Test Domain:
Visit https://crestovillehealth.org to ensure the app loads.

Step 8: Post-Deployment Tasks
Objective: Ensure the app runs smoothly and securely.
Set Up Backups:
Create a cron job for daily backups:
bash

crontab -e

Add:
bash

0 2 * * * cd /var/www/crestoville && /var/www/crestoville/venv/bin/python database_backup.py backup >> /var/log/crestoville_backup.log 2>&1

Monitor Logs:
Check Gunicorn logs:
bash

sudo journalctl -u crestoville.service

Update Admin Password:
Log in as admin@crestovillehealth.org at /admin/dashboard and change the default password.

Test Key Features:
Verify CrestovilleGPT (/crestoville-gpt), Stripe payments, and the CHIES portal.

Troubleshooting
Database Connection Issues: Check DATABASE_URL and network access to Neon.

App Not Starting: Review logs (journalctl -u crestoville) and ensure .env is correct.

SSL Errors: Re-run Certbot or check Nginx config.

This guide deploys your Flask app to Linode, leveraging its existing Neon PostgreSQL database, Gunicorn for serving, and Nginx for proxying, all secured with HTTPS and linked to crestovillehealth.org

