# Run multiple django project with nginx and gunicorn

1. sudo nano /etc/systemd/system/firstsite.socket
```
    [Unit]
    Description=firstsite socket
    
    [Socket]
    ListenStream=/run/firstsite.sock
    
    [Install]
    WantedBy=sockets.target
```
2. sudo nano /etc/systemd/system/secondsite.socket
```
    [Unit]
    Description=secondsite socket
    
    [Socket]
    ListenStream=/run/secondsite.sock
    
    [Install]
    WantedBy=sockets.target
```
3. sudo nano /etc/systemd/system/firstsite.service
```
    [Unit]
    Description=gunicorn daemon
    Requires=firstsite.socket
    After=network.target
    
    [Service]
    User=non-root-user
    Group=www-data
    WorkingDirectory=/home/non-root-user/firstsite/
    ExecStart=/home/non-root-user/firstsite/env/bin/gunicorn \
    --access-logfile - \
    --workers 3 \
    --bind unix:/run/firstsite.sock \
    firstsite.wsgi:application
    
    [Install]
    WantedBy=multi-user.target
```


In most cases, you do not need to create the Unix socket file manually. When you start Gunicorn with the `--bind unix:/run/irstsite.sock` option, Gunicorn will automatically create the Unix socket file specified in the path if it does not already exist.

However, there are a few considerations to keep in mind:

    Permissions: Ensure that the directory where the socket file will be created (/run in this case) has appropriate permissions to allow Gunicorn to create the socket file. Gunicorn needs write access to the directory in order to create the socket file.

    Parent Directories: Ensure that all parent directories of the socket file path exist and have appropriate permissions. Gunicorn will not create parent directories if they do not exist.

    Filesystem: Make sure that the filesystem type supports Unix sockets. Most modern Unix-like operating systems (such as Linux) support Unix sockets without any issues.

4. sudo nano /etc/systemd/system/secondsite.service
```
    [Unit]
    Description=gunicorn daemon
    Requires=secondsite.socket
    After=network.target
    
    [Service]
    User=non-root-user
    Group=www-data
    WorkingDirectory=/home/non-root-user/secondsite/
    ExecStart=/home/non-root-user/secondsite/env/bin/gunicorn \
    --access-logfile - \
    --workers 3 \
    --bind unix:/run/secondsite.sock \
    secondsite.wsgi:application
    
    [Install]
    WantedBy=multi-user.target
```
5. NGINX firstsite.ru
```
        location / {
            proxy_pass http://unix:/run/firstsite.sock; 
            include proxy_params;
        }
```
6. NGINX secondsite.ru
```
        location / {
            proxy_pass http://unix:/run/secondsite.sock; 
            include proxy_params;
        }
```

7. Install Gunicorn
```
pip install gunicorn
```

- sudo systemctl start firstsite.socket
- sudo systemctl enable firstsite.socket
- sudo systemctl status firstsite.socket
- sudo systemctl restart firstsite

- sudo systemctl start secondsite.socket
- sudo systemctl enable secondsite.socket
- sudo systemctl status secondsite.socket
- sudo systemctl restart secondsite

