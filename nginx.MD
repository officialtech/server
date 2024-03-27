# Run multiple django project with nginx and gunicorn

1. sudo nano /etc/systemd/system/firstsite.socket

    [Unit]
    Description=firstsite socket
    
    [Socket]
    ListenStream=/run/firstsite.sock
    
    [Install]
    WantedBy=sockets.target

2. sudo nano /etc/systemd/system/secondsite.socket

    [Unit]
    Description=secondsite socket
    
    [Socket]
    ListenStream=/run/secondsite.sock
    
    [Install]
    WantedBy=sockets.target

3. sudo nano /etc/systemd/system/firstsite.service

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

4. sudo nano /etc/systemd/system/secondsite.service
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

5. NGINX firstsite.ru

        location / {
            proxy_pass http://unix:/run/firstsite.sock; 
            include proxy_params;
        }


6. NGINX secondsite.ru


        location / {
            proxy_pass http://unix:/run/secondsite.sock; 
            include proxy_params;
        }


sudo systemctl start firstsite.socket
sudo systemctl enable firstsite.socket
sudo systemctl status firstsite.socket
sudo systemctl restart firstsite

sudo systemctl start secondsite.socket
sudo systemctl enable secondsite.socket
sudo systemctl status secondsite.socket
sudo systemctl restart secondsite
