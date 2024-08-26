# Custom WordPress Theme Development Environment

This README provides instructions for setting up SSL certificates using `mkcert`, configuring Nginx, and updating your hosts file in a local Docker-based WordPress development environment.

## Prerequisites

Ensure you have the following tools installed:

- [Docker](https://www.docker.com/)
- [mkcert](https://github.com/FiloSottile/mkcert)
- [OpenSSL](https://www.openssl.org/)

## Step 1: Generate SSL Certificates using `mkcert`

1. Install `mkcert` if you haven't already:

```bash
mkcert -install
```

2. Create the certificates for your local domain (e.g., `local.wordpress.test`):

```bash
mkcert local.wordpress.test
```

This will generate two files: `localhost-key.pem` and `localhost-cert.pem`, stored in the `nginx/certs/` directory.

## Step 2: Configure Nginx

1. Open the `default.conf` file located in your Nginx configuration directory. If you haven't set one up, you can typically find or create it at `nginx/default.conf`.

2. Modify the `default.conf` to use SSL:

```nginx
server {
    listen 443 ssl;
    server_name local.wordpress.test;

    ssl_certificate /etc/nginx/certs/localhost-cert.pem;
    ssl_certificate_key /etc/nginx/certs/localhost-key.pem;

    root /var/www/html;

    index index.php index.html index.htm;

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        include fastcgi_params;
        fastcgi_pass wordpress:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }

    error_log /var/log/nginx/error.log;
    access_log /var/log/nginx/access.log;
}

server {
    listen 80;
    server_name local.wordpress.test;
    return 301 https://$host$request_uri;
}
```

3. Ensure the paths to the certificates match where you stored them (`/etc/nginx/certs/`).

4. Reload Nginx to apply the changes:

```bash
docker-compose exec nginx nginx -s reload
```

## Step 3: Update Hosts File

1. Add the following entry to your `hosts` file to map your local domain to `localhost`:

```plaintext
127.0.0.1 local.wordpress.test
```

On Linux or macOS, you can edit the hosts file using:

```bash
sudo vim /etc/hosts
```

On Windows, the hosts file is located at `C:\Windows\System32\drivers\etc\hosts`. You'll need to edit it with Administrator privileges.

## Step 4: Access Your Local WordPress Site

After completing the steps above, you can access your local WordPress site securely via `https://local.wordpress.test`.

## Troubleshooting

- **Nginx Not Starting**: Ensure the certificate paths are correct and that the `nginx/certs/` directory is mounted properly in your Docker configuration.
- **SSL Certificate Issues**: Verify that `mkcert` is properly installed and that the local CA is trusted by your system.

## Additional Notes

- This setup assumes that your Docker Compose configuration includes a service for Nginx and WordPress.
- Make sure that the `nginx/certs/` directory is accessible from within your Nginx container.
