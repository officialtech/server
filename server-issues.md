### nginx: [emerg] open() "/etc/nginx/sites-enabled/test.panomiq.com" failed (40: Too many levels of symbolic links) in /etc/nginx/nginx.conf:64
### nginx: configuration file /etc/nginx/nginx.conf test failed

```
- Solution: always give full path like this
- first delete from site enabled
- sudo ln -s /etc/nginx/sites-available/test.panomiq.com /etc/nginx/sites-enabled/
```

