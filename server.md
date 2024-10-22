### Run ASGI server
- `daphne -b 0.0.0.0 -p 8000 core.asgi:application`
- `uvicorn core.asgi:application --host 0.0.0.0 --port 8000`

### Transcription
1. For faster transcriptions
- `If you have a capable GPU, you can also leverage it for faster transcription by installing torch with CUDA support`
- `pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu117`

2. Required installs
- `pip install openai-whisper ffmpeg-python`

3. Whisper Model selection:
- `"tiny": Fastest but less accurate.`
- `"base": Small but good accuracy for basic tasks.`
- `"small": Moderate size and higher accuracy.`
- `"medium": Even more accurate but requires more processing.`
- `"large": Best accuracy but requires more time and resources.`


### ChatBot
1. Linux Required installation
- `sudo apt update && sudo apt install espeak ffmpeg libespeak1`


### Server
1. Linux Required step before doing anything
- `sudo apt update && sudo apt install python3.11 python3.11-full`
- `sudo apt update && sudo apt install gunicorn nginx`
- `sudo chown -R $USER:$USER /var/www/internal/internal`
- `python3.11 -m venv /var/www/internal/internal/venv`

2. If any permission issue
- `sudo chown -R $USER:$USER /var/www/internal/internal/venv`
- `sudo chown www-data:www-data /run/gunicorn` as per your `gunicorn_internal.service` configuration file

3. Gunicorn Service `/etc/systemd/system/gunicorn_internal.service`

```
[Unit]
Description=gunicorn Internal project
Requires=gunicorn.socket
After=network.target

[Service]
User=ubuntu
Group=www-data
WorkingDirectory=/var/www/internal/internal
ExecStart=/var/www/internal/internal/venv/bin/gunicorn \
          --access-logfile - \
          --workers 3 \
          --bind unix:/run/gunicorn/gunicorn_internal.sock \
          core.wsgi:application

[Install]
WantedBy=multi-user.target
```

5. Gunicorn Socket  `/etc/systemd/system/gunicorn_internal.socket` it's not required

```
Description=gunicorn socket for Internal project (Internal is name of the project)

[Socket]
ListenStream=/run/gunicorn/gunicorn_internal.sock

[Install]
WantedBy=sockets.target
```

6. Nginx Configuration
- `/etc/nginx/sites-available/internal.yoursite.com`

```
server {
server_name internal.yoursite.com  www.internal.yoursite.com;
location = /favicon.ico { access_log off; log_not_found off; }
location /static/ {
    root /var/www/internal/internal;
}
    location / {

            include proxy_params;
            proxy_pass http://unix:/run/gunicorn/gunicorn_internal.sock;
    }
}
```

- `sudo ln -s /etc/nginx/sites-available/internal.yoursite.com /etc/nginx/sites-enabled/`

### Debugging on server

1. Gunicorn and Nginx setup
- `sudo ln -s /etc/nginx/sites-available/internal.yoursite.com /etc/nginx/sites-enabled/`
- `sudo service nginx restart`

- `sudo systemctl enable gunicorn_internal`
- `sudo systemctl start gunicorn_internal`
- `sudo systemctl daemon-reload`

