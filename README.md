# 1. [Django on server](https://github.com/officialtech/server/blob/master/README.md#django-on-server)
# 2. [Wordpress on server](https://github.com/officialtech/server/blob/master/README.md#wordpress-on-server)




# Django on server
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





![Wordpress on server](https://gsvr.com.au/wp-content/uploads/2020/08/WordPress-Server.png)

# Wordpress on server

```bash
sudo apt update
```

### Install, create user and database in mysql

```bash
mysql -u root -p
```
if mysql isn't installed ``` sudo apt install mysql-server ```

```bash
CREATE DATABASE wordpress DEFAULT CHARACTER SET utf8 COLLATE utf8_unicode_ci;
```

```bash
CREATE USER 'wordpressUsername'@'%' IDENTIFIED WITH mysql_native_password BY 'wordpressPassword';
```

```bash
GRANT ALL ON wordpress.* TO 'wordpressUsername'@'%';
```

```bash
FLUSH PRIVILEGES;
```

```bash
EXIT;
or
\q;
```

### Install PHP and or Extentions


```bash
sudo apt install php8.1 libapache2-mod-php8.1
sudo apt install php-curl php-gd php-mbstring php-xml php-xmlrpc php-soap php-intl php-zip
```

You’ll need to restart Apache to load these new extensions.
```bash
sudo systemctl restart apache2
```

### Apache2 configurations

```bash
sudo nano /etc/apache2/sites-available/wordpressWebsite.conf
```

Paste below code:
```bash
<VirtualHost *:80>
        # The ServerName directive sets the request scheme, hostname and port that
        # the server uses to identify itself. This is used when creating
        # redirection URLs. In the context of virtual hosts, the ServerName
        # specifies what hostname must appear in the request's Host: header to
        # match this virtual host. For the default virtual host (this file) this
        # value is not decisive as it is used as a last resort host regardless.
        # However, you must set it for any further virtual host explicitly.
        
        ServerName www.fourthX.com
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/wordpress

        # Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
        # error, crit, alert, emerg.
        # It is also possible to configure the loglevel for particular
        # modules, e.g.
        #LogLevel info ssl:warn

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

        # For most configuration files from conf-available/, which are
        # enabled or disabled at a global level, it is possible to
        # include a line for only one particular virtual host. For example the
        # following line enables the CGI configuration for this host only
        # after it has been globally disabled with "a2disconf".
        #Include conf-available/serve-cgi-bin.conf
</VirtualHost>
```


you can enable mod_rewrite so that you can use the WordPress permalink feature:
```bash
sudo a2enmod rewrite
```

Before implementing the changes you’ve made, check to make sure you haven’t made any syntax errors by running the following test:
```bash
sudo apache2ctl configtest
sudo systemctl restart apache2
```

### Download Wordpress in /tmp directory

```bash
cd /tmp

- Download the latest compressed wordpress release:
curl -O https://wordpress.org/latest.tar.gz

- Extract the compressed file:
tar xzvf latest.tar.gz

cp /tmp/wordpress/wp-config-sample.php /tmp/wordpress/wp-config.php
mkdir /tmp/wordpress/wp-content/upgrade

- Copy the entire contents of the directory into your document root:
sudo cp -a /tmp/wordpress/. /var/www/wordpress
```

### Configure Wordpress directory

Ownership of all the files to the www-data user and group. This is the user that the Apache web server runs as, and Apache will need to be able to read and write WordPress files in order to serve the website and perform automatic updates. Update the ownership with the ```chown``` command which allows you to modify file ownership. Be sure to point to your server’s relevant directory:
```bash
sudo chown -R www-data:www-data /var/www/wordpress
sudo find /var/www/wordpress/ -type d -exec chmod 750 {} \;
sudo find /var/www/wordpress/ -type f -exec chmod 640 {} \;
```

grab secure values from the WordPress secret key generator:
```bash
curl -s https://api.wordpress.org/secret-key/1.1/salt/
```
Copy the output of ```curl```

Open:
```bash
sudo nano /var/www/wordpress/wp-config.php
```

Replace the below code with output copied above:
```bash
define('AUTH_KEY',         'put your unique phrase here');
define('SECURE_AUTH_KEY',  'put your unique phrase here');
define('LOGGED_IN_KEY',    'put your unique phrase here');
define('NONCE_KEY',        'put your unique phrase here');
define('AUTH_SALT',        'put your unique phrase here');
define('SECURE_AUTH_SALT', 'put your unique phrase here');
define('LOGGED_IN_SALT',   'put your unique phrase here');
define('NONCE_SALT',       'put your unique phrase here');
```

#### Replace these credentials as well:
The other change you need to make is to set the method that WordPress should use to write to the filesystem. Since you’ve given the web server permission to write where it needs to, you can explicitly set the filesystem method to “direct”. Failure to set this with your current settings would result in WordPress prompting for FTP credentials when performing some actions.
```bash
// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define( 'DB_NAME', 'CHANGE THIS' );

/** MySQL database username */
define( 'DB_USER', 'CHANGE THIS' );

/** MySQL database password */
define( 'DB_PASSWORD', 'CHANGE THIS' );

/** MySQL hostname */
define( 'DB_HOST', 'localhost' );

/** Database Charset to use in creating database tables. */
define( 'DB_CHARSET', 'utf8' );

/** The Database Collate type. Don't change this if in doubt. */
define( 'DB_COLLATE', '' );

define('FS_METHOD', 'direct');
```

```bash
sudo systemctl restart apache2
```

### Completed: Now Check your domain on browser



## Errors

E1. 
```bash
Your PHP installation appears to be missing the MySQL extension which is required by WordPress.
Please check that the mysqli PHP extension is installed and enabled.
```

E1.A: 
```bash
sudo apt install php-mysql
systemctl restart apache2
```

