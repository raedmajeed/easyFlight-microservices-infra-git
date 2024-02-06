server {
listen 80;
listen [::]:80;
server_name easyflight.tech www.easyflight.tech;

    # Redirect HTTP to HTTPS
    return 301 https://$host$request_uri;

    # Additional HTTP configurations...
    access_log /var/log/nginx/host.access.log main;
    # You can keep the existing error_log configuration if needed
    # error_log /var/log/nginx/error.log;
}

server {
listen 443 ssl;
listen [::]:443 ssl;
server_name easyflight.tech www.easyflight.tech;

    ssl_certificate /etc/nginx/ssl/fullchain.pem;
    ssl_certificate_key /etc/nginx/ssl/privkey.pem;

    # Enable SSL protocols and ciphers (adjust as needed)
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers 'TLS_AES_128_GCM_SHA256:TLS_AES_256_GCM_SHA384:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384';
    # Enable HSTS to enforce HTTPS (optional but recommended)
    add_header Strict-Transport-Security "max-age=31536000" always;

    # Access and error logs for HTTPS
    access_log /var/log/nginx/ssl_access.log main;
    error_log /var/log/nginx/ssl_error.log;



    location / {
        proxy_pass http://api-gateway:9292;
        # Additional proxy settings...
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    error_page 500 502 503 504 /50x.html;
    location = /50x.html {
        root /usr/share/nginx/html;
    }
}