# server


### Step 1: Update System Packages

Ensure your server's package list is up to date by running:

```bash
sudo apt update
```

### Step 2: Install Apache and Mod_wsgi

Install Apache and the mod_wsgi module for hosting Python applications:

```bash
sudo apt install apache2 libapache2-mod-wsgi-py3
```

### Step 3: Create a Virtual Environment

It's a good practice to use a virtual environment for your Django project. If you haven't already, create and activate a virtual environment for your project:

```bash
cd /var/www/dir_name  # Navigate to your project directory
python3 -m venv venv  # Create a virtual environment
source venv/bin/activate  # Activate the virtual environment
```

### Step 4: Install Required Packages

Install the required packages for your Django project using pip. Typically, this includes Django itself and any additional packages you've used:

```bash
pip install django  # Replace with your project's requirements
```

### Step 5: Configure Your Django Project

Configure your Django project settings, including the `ALLOWED_HOSTS` setting in your Django project's `settings.py` file to include your domain:

```python
ALLOWED_HOSTS = ['backend.bioemr.com', 'your_server_ip']
```

### Step 6: Collect Static Files

Collect your Django project's static files:

```bash
python manage.py collectstatic
```

### Step 7: Configure Apache

Create a new Apache configuration file for your Django project:

```bash
sudo nano /etc/apache2/sites-available/backend.bioemr.com.conf
```

Add the following configuration to the file, replacing `/var/www/dir_name` with your project's directory path and `your_project.wsgi` with your project's WSGI file:

```apache
<VirtualHost *:80>
    ServerAdmin admin@backend.bioemr.com
    ServerName backend.bioemr.com
    DocumentRoot /var/www/dir_name

    WSGIDaemonProcess your_project python-home=/var/www/dir_name/venv python-path=/var/www/dir_name
    WSGIProcessGroup your_project
    WSGIScriptAlias / /var/www/dir_name/your_project.wsgi

    Alias /static/ /var/www/dir_name/static/
    Alias /media/ /var/www/dir_name/media/

    <Directory /var/www/dir_name>
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

Save the file and exit the text editor.

### Step 8: Enable the Apache Site

Enable the site configuration and disable the default Apache site:

```bash
sudo a2ensite backend.bioemr.com
sudo a2dissite 000-default.conf  # Disable the default site if it's enabled
```

### Step 9: Test Apache Configuration

Test the Apache configuration to check for any syntax errors:

```bash
sudo apache2ctl configtest
```

If you see "Syntax OK," the configuration is correct.

### Step 10: Restart Apache

Restart Apache to apply the changes:

```bash
sudo systemctl restart apache2
```

### Step 11: Configure DNS

Ensure that your domain `backend.bioemr.com` points to your server's IP address. This is typically done through your domain registrar's DNS settings.

### Step 12: Update Your Server's Firewall

If you have a firewall enabled (e.g., UFW), make sure it allows traffic on port 80 (HTTP):

```bash
sudo ufw allow 80/tcp
```

### Step 13: Access Your Django Project

Now that your Django project is configured and your server is set up, you should be able to access your project at `http://backend.bioemr.com`. Your Django website should be up and running!

Remember to keep your server and Django project secure by regularly updating packages, configuring HTTPS (SSL/TLS), and following best practices for web application security.

If you encounter any issues during this setup or have further questions, please feel free to ask for assistance.
