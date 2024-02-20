# Nginx RTMP Server with HLS and Authentication Project

## Overview

The Nginx RTMP server with HLS and authentication is a project aimed at setting up a multimedia streaming server using Nginx with RTMP support. RTMP (Real-Time Messaging Protocol) is a protocol for transmitting audio, video, and data over the internet in real-time. HLS (HTTP Live Streaming) is a streaming protocol developed by Apple for delivering multimedia content over HTTP. This project integrates Nginx's RTMP module with HLS streaming capability and adds authentication for secure access.

## How it Works

1. **RTMP Server**: Nginx is configured with the RTMP module to act as a streaming server. It listens on the RTMP port (1935) for incoming live streams.
2. **HLS Conversion**: When a live stream is received, Nginx converts it into HLS format on-the-fly. HLS breaks the stream into smaller segments and serves them via HTTP.
3. **Authentication**: The server is configured to require authentication for accessing the HLS streams. Users must provide a username and password to view the content.

## Applications

1. **Live Streaming**: Ideal for broadcasting live events, sports matches, concerts, and conferences over the internet.
2. **Video On Demand (VOD)**: Supports delivery of pre-recorded video content in HLS format, making it suitable for video-on-demand services.
3. **Surveillance Systems**: Can be used to stream live video feeds from security cameras, enabling remote monitoring and surveillance.

## Benefits

1. **Cross-Platform Compatibility**: HLS streams can be played on a wide range of devices and platforms, including desktops, mobile devices, smart TVs, and set-top boxes.
2. **Scalability**: Nginx's ability to handle multiple concurrent connections makes it suitable for scaling the streaming server to accommodate large audiences.
3. **Security**: Integration of authentication ensures that only authorized users can access the streaming content, enhancing security and preventing unauthorized access.
4. **Adaptive Bitrate Streaming**: HLS supports adaptive bitrate streaming, allowing users to stream content at the best quality based on their network conditions.

## Limitations

1. **Latency**: HLS introduces some latency due to the segmentation of the stream into chunks, which may not be suitable for applications requiring real-time interaction.
2. **Resource Intensive**: On-the-fly conversion of RTMP streams into HLS format can be resource-intensive, especially for high-resolution streams or when serving a large number of concurrent users.
3. **Complexity**: Setting up and configuring the server requires technical expertise, and troubleshooting issues may require familiarity with Nginx configuration and RTMP streaming protocols.
4. **Browser Support**: While HLS is widely supported, compatibility issues may arise with older browsers or devices, limiting the reach of the streaming content.

---

This project provides a robust solution for streaming multimedia content over the internet, offering scalability, security, and compatibility with a wide range of devices. However, it's essential to consider the limitations and complexities involved in its setup and operation.

---

# Setting up Nginx RTMP Server with HLS and Authentication

This guide provides step-by-step instructions for creating an Nginx RTMP server with HLS (HTTP Live Streaming) support and authentication using Nginx's `libnginx-mod-rtmp` module.

## Prerequisites

- Ubuntu Linux server (or similar distribution)
- `sudo` privileges
- Basic knowledge of Linux command line

## Installation

### Step 1: Install `libnginx-mod-rtmp`

Update the package index and install the `libnginx-mod-rtmp` package:

```bash
sudo apt update
sudo apt install libnginx-mod-rtmp
```

### Step 2: Configure Nginx

Open the Nginx configuration file for editing:

```bash
sudo nano /etc/nginx/nginx.conf
```

Replace the contents with the following configuration:

```nginx
# Nginx configuration for RTMP and HLS

user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
    worker_connections 768;
}

http {
    sendfile on;
    tcp_nopush on;
    types_hash_max_size 2048;
    include /etc/nginx/mime.types;
    default_type application/octet-stream;
    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;
    gzip on;
    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
}

rtmp {
    server {
        listen 1935;
        chunk_size 4096;
        allow publish all;
        allow play all;

        application live {
            live on;
            record off;
            hls on;
            hls_path /var/www/html/stream/hls;
            hls_fragment 3;
            hls_playlist_length 60;
        }
    }
}
```

### Step 3: Set Up Authentication

Install Apache utilities for password encryption:

```bash
sudo apt-get install apache2-utils
```

Create a username and password for authentication:

```bash
sudo htpasswd -c /etc/nginx/.htpasswd username
```

Replace `username` with your desired username. You will be prompted to enter and confirm the password.

### Step 4: Configure Nginx Sites

Create a new configuration file for Nginx:

```bash
sudo nano /etc/nginx/sites-available/rtmp
```

Add the following configuration:

```nginx
server {
    listen 8080;
    server_name localhost;

    location /stat {
        rtmp_stat all;
        rtmp_stat_stylesheet stat.xsl;
    }
    location /stat.xsl {
        root /var/www/html/rtmp;
    }

    location /control {
        rtmp_control all;
    }
}

server {
    listen 8088;

    location / {
        auth_basic "Restricted";
        auth_basic_user_file /etc/nginx/.htpasswd;

        add_header Access-Control-Allow-Origin *;
        root /var/www/html/stream;
    }
}

types {
    application/dash+xml mpd;
}
```

### Step 5: Configure Access Control

Create necessary directories and permissions:

```bash
sudo mkdir /var/www/html/rtmp
sudo gunzip -c /usr/share/doc/libnginx-mod-rtmp/examples/stat.xsl.gz > /var/www/html/rtmp/stat.xsl
```

### Step 6: Firewall Configuration

Allow necessary ports in the firewall:

```bash
sudo ufw allow from your_ip_address to any port http-alt
sudo ufw allow 1935/tcp
sudo ufw allow 8088/tcp
```

Replace `your_ip_address` with your actual IP address. You can find your IP address by running `hostname -I`.

### Step 7: Enable Configuration

Create a symbolic link for the Nginx configuration file:

```bash
sudo ln -s /etc/nginx/sites-available/rtmp /etc/nginx/sites-enabled/rtmp
```

### Step 8: Reload Nginx

Reload Nginx to apply the changes:

```bash
sudo systemctl reload nginx.service
```

## Conclusion

You have successfully set up an Nginx RTMP server with HLS streaming and authentication. You can now start publishing and playing live video streams using this setup.

---
