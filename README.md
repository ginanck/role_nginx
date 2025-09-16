# Ansible Role: Nginx

[![Ansible Role](https://img.shields.io/ansible/role/nginx.svg)](https://galaxy.ansible.com/role_nginx) [![License](https://img.shields.io/badge/license-BSD-brightgreen.svg)](https://opensource.org/licenses/BSD-2-Clause)

This comprehensive Ansible role installs and configures Nginx web server for various use cases including default servers, virtual hosts, HTTP load balancers, stream load balancers, and static file servers. The role provides flexible configuration options for both simple deployments and complex enterprise environments.

## Features

- **Multi-platform support**: RedHat and Debian family distributions
- **Default server configuration**: Basic HTTP/HTTPS server setup
- **Virtual host management**: Multiple website hosting with SSL/TLS support
- **Load balancing**: HTTP and TCP/UDP stream load balancing
- **Static file serving**: Optimized static content delivery
- **Security hardening**: SELinux integration and firewall management
- **Performance tuning**: Worker process and connection optimization
- **SSL/TLS configuration**: Comprehensive SSL/TLS support with modern ciphers
- **Logging configuration**: Customizable access and error logging

## Requirements

- **Ansible**: 2.9 or higher
- **Python**: 3.6 or higher on managed nodes
- **Supported Operating Systems**:
  - **RHEL Family**: CentOS 7+, RHEL 7+, Rocky Linux 8+, AlmaLinux 8+
  - **Debian Family**: Ubuntu 18.04+, Debian 9+

## Role Variables

### Core System Configuration

- `nginx_service_packages`: (list, default: ["nginx"]) List of Nginx packages to install
- `nginx_service_name`: (string, default: "nginx") Name of the Nginx service
- `nginx_service_pid`: (string, default: "/run/nginx.pid") Path to Nginx PID file
- `nginx_service_config_path`: (string, default: "/etc/nginx") Path to Nginx configuration directory
- `nginx_service_root_path`: (string, default: "/var/www/html") Default document root path
- `nginx_service_ssl_path`: (string, default: "/opt/nginx/ssl") Path for SSL certificates and keys

### Worker Process Configuration

- `nginx_service_worker_processes`: (string, default: "auto") Number of worker processes
- `nginx_service_worker_connections`: (int, default: 1024) Maximum connections per worker
- `nginx_service_worker_rlimit_nofile`: (int, default: 4096) File descriptor limit for workers

### Network and Performance Settings

- `nginx_service_sendfile`: (string, default: "on") Enable sendfile for efficient file transfers
- `nginx_service_tcp_nopush`: (string, default: "on") Enable TCP_NOPUSH socket option
- `nginx_service_tcp_nodelay`: (string, default: "on") Enable TCP_NODELAY socket option
- `nginx_service_keepalive_timeout`: (int, default: 65) Keep-alive connection timeout in seconds
- `nginx_service_types_hash_max_size`: (int, default: 2048) Maximum size of MIME types hash table
- `nginx_service_include_mime_type`: (string, default: "{{ nginx_service_config_path }}/mime.types") Path to MIME types file
- `nginx_service_default_type`: (string, default: "application/octet-stream") Default MIME type

### SSL/TLS Configuration

- `nginx_ssl_session_cache_http`: (string, default: "shared:HTTPSSLCache:10m") SSL session cache for HTTP
- `nginx_ssl_session_cache_stream`: (string, default: "shared:StreamSSLCache:10m") SSL session cache for stream

### Logging Configuration

- `nginx_service_error_log`: (string, default: "/var/log/nginx/error.log") Path to main error log
- `nginx_httpd_access_log`: (string, default: "/var/log/nginx/httpd_access.log") HTTP access log path
- `nginx_httpd_error_log`: (string, default: "/var/log/nginx/httpd_error.log") HTTP error log path
- `nginx_stream_access_log`: (string, default: "/var/log/nginx/stream_access.log") Stream access log path
- `nginx_stream_error_log`: (string, default: "/var/log/nginx/stream_error.log") Stream error log path
- `nginx_static_access_log`: (string, default: "/var/log/nginx/static_access.log") Static files access log path
- `nginx_static_error_log`: (string, default: "/var/log/nginx/static_error.log") Static files error log path

### Log Format Configuration

- `nginx_httpd_log_format`: (dict) HTTP access log format configuration
- `nginx_stream_log_format`: (dict) Stream access log format configuration  
- `nginx_static_log_format`: (dict) Static files access log format configuration

### Security Configuration

- `nginx_selinux_enabled`: (bool, default: true) Enable SELinux integration
- `nginx_selinux_ports`: (list, default: [80, 443, 6443]) Ports to configure in SELinux
- `nginx_firewall_disable`: (bool, default: true) Disable system firewall

### System Optimization

- `nginx_sysctl_conf_file`: (string, default: "99-nginx.conf") Sysctl configuration file name
- `nginx_sysctl_parameters`: (list) System kernel parameters to optimize

### Server Configuration Arrays

- `nginx_default_server`: (list, default: []) Default server configurations
- `nginx_vhost_servers`: (list, default: []) Virtual host server configurations
- `nginx_static_file_servers`: (list, default: []) Static file server configurations
- `nginx_http_loadbalancers`: (list, default: []) HTTP load balancer configurations
- `nginx_stream_loadbalancers`: (list, default: []) Stream load balancer configurations

### Platform-Specific Variables

#### RedHat Family
- `nginx_service_packages`: (list, default: ["nginx", "nginx-all-modules"]) Enhanced package list for RedHat
- `selinux_packages`: (list, default: ["policycoreutils-python-utils", "python3-libselinux", "python3-policycoreutils"]) SELinux packages for RedHat

#### Debian Family  
- `nginx_service_packages`: (list, default: ["nginx"]) Basic package list for Debian
- `selinux_packages`: (list, default: ["python3-selinux", "selinux-utils", "policycoreutils"]) SELinux packages for Debian

## Dependencies

None.

## Example Playbooks

### Basic Nginx Installation

```yaml
---
- name: Install basic Nginx server
  hosts: webservers
  become: true
  roles:
    - role: role_nginx
      vars:
        # Core configuration
        nginx_service_worker_processes: 2
        nginx_service_worker_connections: 2048
        nginx_service_keepalive_timeout: 30
        
        # Basic default server
        nginx_default_server:
          - name: "default_http"
            enabled: true
            listen:
              - name: "default_server"
                ip: ""
                ipv6: "[::]"
                port: 80
            server_name: "_"
            root: "/var/www/html"
            locations:
              - path: "/"
                config:
                  - "index index.html index.htm;"
                  - "try_files $uri $uri/ =404;"
            error_pages:
              - page: "/404.html"
                codes: [404]
              - page: "/50x.html"
                codes: [500, 502, 503, 504]
```

### Virtual Host Configuration with SSL

```yaml
---
- name: Configure virtual hosts with SSL
  hosts: webservers
  become: true
  roles:
    - role: role_nginx
      vars:
        # Enhanced worker configuration
        nginx_service_worker_processes: auto
        nginx_service_worker_connections: 4096
        nginx_service_worker_rlimit_nofile: 8192
        
        # Performance optimization
        nginx_service_sendfile: "on"
        nginx_service_tcp_nopush: "on"
        nginx_service_tcp_nodelay: "on"
        nginx_service_keepalive_timeout: 65
        
        # SSL session caching
        nginx_ssl_session_cache_http: "shared:HTTPSSLCache:50m"
        
        # SELinux and security
        nginx_selinux_enabled: true
        nginx_selinux_ports: [80, 443, 8080, 8443]
        nginx_firewall_disable: true
        
        # System optimization
        nginx_sysctl_parameters:
          - name: net.ipv4.ip_forward
            value: 1
          - name: net.ipv6.conf.all.forwarding
            value: 1
          - name: net.core.somaxconn
            value: 32768
          - name: vm.swappiness
            value: 0
        
        # Virtual hosts configuration
        nginx_vhost_servers:
          - name: "corporate_website"
            enabled: true
            server_name: "www.example.com example.com"
            root: "/var/www/corporate"
            
            # Multi-port listening with SSL
            listen:
              - name: "default_server"
                port: 80
                ip: ""
                ipv6: "[::]"
              - name: "ssl_server"
                port: 443
                ip: ""
                ipv6: "[::]"
                ssl: true
                http2: true
            
            # SSL/TLS configuration
            tls:
              ssl_certificate: "/etc/ssl/certs/corporate.crt"
              ssl_certificate_key: "/etc/ssl/private/corporate.key"
              ssl_session_cache: "{{ nginx_ssl_session_cache_http }}"
              ssl_session_timeout: "1d"
              ssl_ciphers: "ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256"
              ssl_prefer_server_ciphers: "off"
            
            # Advanced logging
            access_log: "/var/log/nginx/corporate_access.log"
            error_log: "/var/log/nginx/corporate_error.log"
            client_max_body_size: "50m"
            
            # Backend integration
            upstreams:
              - name: "app_backend"
                lb_method: "least_conn"
                servers:
                  - address: "192.168.1.10:8080"
                    weight: 2
                  - address: "192.168.1.11:8080"
                    weight: 1
            
            # Location blocks
            locations:
              - path: "/"
                config:
                  - "index index.html index.php;"
                  - "try_files $uri $uri/ /index.php?$args;"
              
              - path: "/api"
                proxy_pass: "app_backend"
                config:
                  - "proxy_connect_timeout 30s;"
                  - "proxy_read_timeout 60s;"
                  - "proxy_send_timeout 60s;"
                  - "proxy_set_header Host $host;"
                  - "proxy_set_header X-Real-IP $remote_addr;"
                  - "proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;"
                  - "proxy_set_header X-Forwarded-Proto $scheme;"
              
              - path: "/static"
                config:
                  - "alias /var/www/static;"
                  - "expires 30d;"
                  - "add_header Cache-Control public;"
                  - "access_log off;"
            
            # Error pages
            error_pages:
              - codes: [404]
                page: "/404.html"
                location:
                  - "internal;"
              - codes: [500, 502, 503, 504]
                page: "/50x.html"
                location:
                  - "internal;"
            
            # Security headers
            extra_config:
              - "add_header X-Frame-Options SAMEORIGIN;"
              - "add_header X-Content-Type-Options nosniff;"
              - "add_header X-XSS-Protection \"1; mode=block\";"
              - "add_header Strict-Transport-Security \"max-age=31536000; includeSubDomains\" always;"
```

### HTTP Load Balancer Configuration

```yaml
---
- name: Configure HTTP load balancers
  hosts: loadbalancers
  become: true
  roles:
    - role: role_nginx
      vars:
        # High-performance worker configuration
        nginx_service_worker_processes: auto
        nginx_service_worker_connections: 8192
        nginx_service_worker_rlimit_nofile: 16384
        
        # Performance tuning
        nginx_service_keepalive_timeout: 30
        nginx_service_types_hash_max_size: 4096
        
        # Custom log formats for load balancing
        nginx_httpd_log_format:
          http:
            - '$remote_addr - $remote_user [$time_local] "$request" '
            - '$status $body_bytes_sent "$http_referer" '
            - '"$http_user_agent" "$http_x_forwarded_for" '
            - '$request_time $upstream_response_time $upstream_addr'
        
        # HTTP load balancers
        nginx_http_loadbalancers:
          - name: "web_cluster"
            enabled: true
            lb_method: "least_conn"
            
            listen:
              - name: "web_lb"
                ip: ""
                ipv6: "[::]"
                port: 80
              - name: "web_lb_ssl"
                ip: ""
                ipv6: "[::]"
                port: 443
                ssl: true
                http2: true
            
            server_name: "app.example.com"
            keepalive: 64
            
            # Backend servers
            upstream_servers:
              - address: "192.168.1.20:8080"
                weight: 3
                max_fails: 2
                fail_timeout: "30s"
              - address: "192.168.1.21:8080"
                weight: 2
                max_fails: 2
                fail_timeout: "30s"
              - address: "192.168.1.22:8080"
                weight: 1
                backup: true
            
            # Proxy settings
            proxy_connect_timeout: "10s"
            proxy_read_timeout: "120s"
            proxy_send_timeout: "120s"
            client_max_body_size: "100m"
            
            # Location handling
            locations:
              - path: "/"
                proxy_pass: "web_cluster"
                config:
                  - "proxy_set_header Host $host;"
                  - "proxy_set_header X-Real-IP $remote_addr;"
                  - "proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;"
                  - "proxy_set_header X-Forwarded-Proto $scheme;"
                  - "proxy_redirect off;"
              
              - path: "/health"
                config:
                  - "access_log off;"
                  - "return 200 'healthy\\n';"
                  - "add_header Content-Type text/plain;"
            
            # SSL configuration
            tls:
              ssl_certificate: "/etc/ssl/certs/app.crt"
              ssl_certificate_key: "/etc/ssl/private/app.key"
              ssl_session_cache: "{{ nginx_ssl_session_cache_http }}"
              ssl_session_timeout: "1d"
              ssl_ciphers: "ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256"
              ssl_prefer_server_ciphers: "off"
            
            error_pages:
              - page: "/maintenance.html"
                codes: [502, 503, 504]
```

### Stream Load Balancer for Database Clustering

```yaml
---
- name: Configure stream load balancers for databases
  hosts: db_proxies
  become: true
  roles:
    - role: role_nginx
      vars:
        # Stream-optimized configuration
        nginx_service_worker_processes: auto
        nginx_service_worker_connections: 2048
        
        # Stream logging
        nginx_stream_log_format:
          stream:
            - '$remote_addr [$time_local] '
            - '$protocol $status $bytes_sent $bytes_received '
            - '$session_time "$upstream_addr" $upstream_connect_time'
        
        # SELinux configuration for database ports
        nginx_selinux_enabled: true
        nginx_selinux_ports: [3306, 5432, 6379, 27017]
        
        # Stream load balancers
        nginx_stream_loadbalancers:
          - name: "mysql_cluster"
            enabled: true
            lb_method: "hash"
            
            listen:
              - name: "mysql_proxy"
                ip: ""
                ipv6: "[::]"
                port: 3306
            
            # MySQL backend servers
            upstream_servers:
              - address: "192.168.2.10:3306"
                weight: 2
                max_fails: 3
                fail_timeout: "10s"
              - address: "192.168.2.11:3306"
                weight: 2
                max_fails: 3
                fail_timeout: "10s"
              - address: "192.168.2.12:3306"
                weight: 1
                backup: true
            
            proxy_timeout: "300s"
            proxy_connect_timeout: "5s"
          
          - name: "postgres_cluster"
            enabled: true
            lb_method: "least_conn"
            
            listen:
              - name: "postgres_proxy"
                ip: ""
                ipv6: "[::]"
                port: 5432
                ssl: true
            
            # PostgreSQL backend servers
            upstream_servers:
              - address: "192.168.2.20:5432"
              - address: "192.168.2.21:5432"
            
            proxy_timeout: "600s"
            ssl: true
            
            # SSL configuration for PostgreSQL
            tls:
              ssl_certificate: "/etc/ssl/certs/postgres.crt"
              ssl_certificate_key: "/etc/ssl/private/postgres.key"
              ssl_session_cache: "{{ nginx_ssl_session_cache_stream }}"
              ssl_session_timeout: "1h"
              ssl_ciphers: "HIGH:!aNULL:!MD5"
              ssl_prefer_server_ciphers: true
          
          - name: "redis_cluster"
            enabled: true
            lb_method: "hash"
            
            listen:
              - name: "redis_proxy"
                ip: ""
                ipv6: "[::]"
                port: 6379
            
            # Redis backend servers
            upstream_servers:
              - address: "192.168.2.30:6379"
              - address: "192.168.2.31:6379"
              - address: "192.168.2.32:6379"
            
            proxy_timeout: "60s"
            proxy_connect_timeout: "3s"
```

### Static File Server with CDN Features

```yaml
---
- name: Configure static file servers
  hosts: cdn_servers
  become: true
  roles:
    - role: role_nginx
      vars:
        # Performance optimization for static content
        nginx_service_worker_processes: auto
        nginx_service_worker_connections: 4096
        nginx_service_sendfile: "on"
        nginx_service_tcp_nopush: "on"
        nginx_service_tcp_nodelay: "on"
        
        # Static file logging
        nginx_static_log_format:
          static:
            - '$remote_addr - $remote_user [$time_local] "$request" '
            - '$status $body_bytes_sent "$http_referer" '
            - '"$http_user_agent" $request_time $upstream_cache_status'
        
        # Static file servers
        nginx_static_file_servers:
          - name: "cdn_server"
            enabled: true
            
            listen:
              - port: 80
                ip: ""
                name: "default_server"
              - port: 443
                ssl: true
                http2: true
                ip: ""
            
            server_name: "cdn.example.com static.example.com"
            root: "/var/www/static"
            index: "index.html index.htm"
            autoindex: false
            
            # Logging configuration
            access_log: "/var/log/nginx/cdn_access.log"
            error_log: "/var/log/nginx/cdn_error.log"
            client_max_body_size: "500M"
            
            # SSL/TLS configuration
            tls:
              ssl_certificate: "/etc/ssl/certs/cdn.crt"
              ssl_certificate_key: "/etc/ssl/private/cdn.key"
              ssl_session_cache: "shared:CDNSSLCache:50m"
              ssl_session_timeout: "1d"
              ssl_ciphers: "ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256"
              ssl_prefer_server_ciphers: "off"
            
            # Optimized location blocks
            locations:
              - path: "/"
                try_files: "$uri $uri/ =404"
                expires: "7d"
                add_headers:
                  - name: "Cache-Control"
                    value: "public, immutable"
                  - name: "X-Content-Type-Options"
                    value: "nosniff"
                    always: true
                  - name: "X-Frame-Options"
                    value: "DENY"
                    always: true
              
              - path: "/images"
                alias: "/var/www/images"
                autoindex: false
                config:
                  - "expires 30d;"
                  - "add_header Cache-Control \"public, immutable\";"
                  - "add_header Vary Accept-Encoding;"
                  - "gzip_static on;"
                  - "access_log off;"
              
              - path: "/downloads"
                alias: "/var/www/downloads"
                autoindex: true
                config:
                  - "expires 1d;"
                  - "add_header Content-Disposition 'attachment';"
                  - "limit_rate 1m;"
              
              - path: "/assets"
                config:
                  - "expires max;"
                  - "add_header Cache-Control \"public, immutable\";"
                  - "gzip_static on;"
                  - "brotli_static on;"
                  - "access_log off;"
            
            # Error pages
            error_pages:
              - codes: [404]
                page: "/404.html"
              - codes: [500, 502, 503, 504]
                page: "/50x.html"
                location:
                  - "root /usr/share/nginx/html;"
                  - "internal;"
            
            # Additional optimization
            extra_config:
              - "gzip on;"
              - "gzip_vary on;"
              - "gzip_min_length 1024;"
              - "gzip_types text/plain text/css text/xml text/javascript application/javascript application/xml+rss application/json;"
              - "open_file_cache max=10000 inactive=5m;"
              - "open_file_cache_valid 2m;"
              - "open_file_cache_min_uses 1;"
              - "open_file_cache_errors on;"
```

### Complete Production Environment

```yaml
---
- name: Complete production Nginx deployment
  hosts: production_servers
  become: true
  roles:
    - role: role_nginx
      vars:
        # Production-grade worker configuration
        nginx_service_worker_processes: auto
        nginx_service_worker_connections: 8192
        nginx_service_worker_rlimit_nofile: 65536
        
        # Production performance settings
        nginx_service_sendfile: "on"
        nginx_service_tcp_nopush: "on"
        nginx_service_tcp_nodelay: "on"
        nginx_service_keepalive_timeout: 65
        nginx_service_types_hash_max_size: 4096
        
        # Production SSL caching
        nginx_ssl_session_cache_http: "shared:HTTPSSLCache:100m"
        nginx_ssl_session_cache_stream: "shared:StreamSSLCache:50m"
        
        # Production logging paths
        nginx_service_error_log: "/var/log/nginx/error.log warn"
        nginx_httpd_access_log: "/var/log/nginx/access.log"
        nginx_httpd_error_log: "/var/log/nginx/http_error.log"
        nginx_stream_access_log: "/var/log/nginx/stream_access.log"
        nginx_stream_error_log: "/var/log/nginx/stream_error.log"
        nginx_static_access_log: "/var/log/nginx/static_access.log"
        nginx_static_error_log: "/var/log/nginx/static_error.log"
        
        # Security configuration
        nginx_selinux_enabled: true
        nginx_selinux_ports: [80, 443, 8080, 8443, 3306, 5432, 6379]
        nginx_firewall_disable: false
        
        # System optimization for production
        nginx_sysctl_conf_file: "99-nginx-production.conf"
        nginx_sysctl_parameters:
          - name: net.ipv4.ip_forward
            value: 1
          - name: net.ipv6.conf.all.forwarding
            value: 1
          - name: net.bridge.bridge-nf-call-iptables
            value: 1
          - name: net.bridge.bridge-nf-call-ip6tables
            value: 1
          - name: net.core.somaxconn
            value: 65535
          - name: net.core.netdev_max_backlog
            value: 5000
          - name: net.ipv4.tcp_max_syn_backlog
            value: 8192
          - name: net.ipv4.tcp_fin_timeout
            value: 30
          - name: vm.swappiness
            value: 0
          - name: fs.file-max
            value: 2097152
        
        # Default server (disabled in production)
        nginx_default_server:
          - name: "default_deny"
            enabled: true
            listen:
              - name: "default_server"
                ip: ""
                ipv6: "[::]"
                port: 80
            server_name: "_"
            locations:
              - path: "/"
                config:
                  - "return 444;"
        
        # Production virtual hosts
        nginx_vhost_servers:
          - name: "main_application"
            enabled: true
            server_name: "app.production.com"
            root: "/var/www/app"
            
            listen:
              - port: 443
                ssl: true
                http2: true
                ip: ""
                ipv6: "[::]"
            
            tls:
              ssl_certificate: "/etc/ssl/certs/production.crt"
              ssl_certificate_key: "/etc/ssl/private/production.key"
              ssl_session_cache: "{{ nginx_ssl_session_cache_http }}"
              ssl_session_timeout: "1d"
              ssl_ciphers: "ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384"
              ssl_prefer_server_ciphers: "off"
            
            access_log: "/var/log/nginx/app_access.log"
            error_log: "/var/log/nginx/app_error.log"
            client_max_body_size: "100m"
            
            upstreams:
              - name: "app_backend"
                lb_method: "least_conn"
                servers:
                  - address: "10.0.1.10:8080"
                    weight: 2
                  - address: "10.0.1.11:8080"
                    weight: 2
                  - address: "10.0.1.12:8080"
                    weight: 1
                    backup: true
            
            locations:
              - path: "/"
                proxy_pass: "app_backend"
                config:
                  - "proxy_connect_timeout 30s;"
                  - "proxy_read_timeout 300s;"
                  - "proxy_send_timeout 300s;"
                  - "proxy_set_header Host $host;"
                  - "proxy_set_header X-Real-IP $remote_addr;"
                  - "proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;"
                  - "proxy_set_header X-Forwarded-Proto $scheme;"
                  - "proxy_buffering on;"
                  - "proxy_buffer_size 128k;"
                  - "proxy_buffers 4 256k;"
                  - "proxy_busy_buffers_size 256k;"
            
            extra_config:
              - "add_header Strict-Transport-Security \"max-age=63072000; includeSubDomains; preload\" always;"
              - "add_header X-Frame-Options DENY always;"
              - "add_header X-Content-Type-Options nosniff always;"
              - "add_header X-XSS-Protection \"1; mode=block\" always;"
              - "add_header Referrer-Policy \"strict-origin-when-cross-origin\" always;"
        
        # Production HTTP load balancer
        nginx_http_loadbalancers:
          - name: "api_cluster"
            enabled: true
            lb_method: "least_conn"
            
            listen:
              - name: "api_lb"
                ip: ""
                port: 443
                ssl: true
                http2: true
            
            server_name: "api.production.com"
            keepalive: 128
            
            upstream_servers:
              - address: "10.0.2.10:8080"
                weight: 3
                max_fails: 2
                fail_timeout: "30s"
              - address: "10.0.2.11:8080"
                weight: 3
                max_fails: 2
                fail_timeout: "30s"
              - address: "10.0.2.12:8080"
                weight: 2
                max_fails: 2
                fail_timeout: "30s"
              - address: "10.0.2.13:8080"
                weight: 1
                backup: true
            
            proxy_connect_timeout: "30s"
            proxy_read_timeout: "300s"
            proxy_send_timeout: "300s"
            client_max_body_size: "50m"
            
            tls:
              ssl_certificate: "/etc/ssl/certs/api.crt"
              ssl_certificate_key: "/etc/ssl/private/api.key"
              ssl_session_cache: "{{ nginx_ssl_session_cache_http }}"
              ssl_session_timeout: "1d"
              ssl_ciphers: "ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256"
              ssl_prefer_server_ciphers: "off"
        
        # Production stream load balancer
        nginx_stream_loadbalancers:
          - name: "database_cluster"
            enabled: true
            lb_method: "hash"
            
            listen:
              - name: "db_proxy"
                ip: ""
                port: 5432
                ssl: true
            
            upstream_servers:
              - address: "10.0.3.10:5432"
                weight: 2
              - address: "10.0.3.11:5432"
                weight: 2
              - address: "10.0.3.12:5432"
                weight: 1
                backup: true
            
            proxy_timeout: "600s"
            proxy_connect_timeout: "10s"
            ssl: true
            
            tls:
              ssl_certificate: "/etc/ssl/certs/database.crt"
              ssl_certificate_key: "/etc/ssl/private/database.key"
              ssl_session_cache: "{{ nginx_ssl_session_cache_stream }}"
              ssl_session_timeout: "4h"
              ssl_ciphers: "HIGH:!aNULL:!MD5"
              ssl_prefer_server_ciphers: true
        
        # Production static file server
        nginx_static_file_servers:
          - name: "static_assets"
            enabled: true
            
            listen:
              - port: 443
                ssl: true
                http2: true
                ip: ""
            
            server_name: "static.production.com"
            root: "/var/www/static"
            index: "index.html"
            autoindex: false
            
            access_log: "/var/log/nginx/static_access.log"
            error_log: "/var/log/nginx/static_error.log"
            client_max_body_size: "1G"
            
            tls:
              ssl_certificate: "/etc/ssl/certs/static.crt"
              ssl_certificate_key: "/etc/ssl/private/static.key"
              ssl_session_cache: "shared:StaticSSL:50m"
              ssl_session_timeout: "1d"
              ssl_ciphers: "ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256"
              ssl_prefer_server_ciphers: "off"
            
            locations:
              - path: "/"
                try_files: "$uri $uri/ =404"
                expires: "30d"
                add_headers:
                  - name: "Cache-Control"
                    value: "public, immutable"
                  - name: "X-Content-Type-Options"
                    value: "nosniff"
                    always: true
            
            extra_config:
              - "gzip on;"
              - "gzip_vary on;"
              - "gzip_min_length 1024;"
              - "gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;"
              - "brotli on;"
              - "brotli_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;"
```

## Detailed Feature Sections

### Default Server Configuration

The default server feature allows you to configure basic HTTP and HTTPS servers with minimal setup. This is ideal for simple websites, testing environments, or as a foundation for more complex configurations.

**Key Features:**
- HTTP and HTTPS support
- IPv4 and IPv6 binding
- Custom error pages
- Basic location blocks
- SSL/TLS termination

**Configuration Structure:**
```yaml
nginx_default_server:
  - name: "unique_name"           # Required: Unique identifier
    enabled: true                 # Required: Enable/disable this server
    listen: []                    # Required: List of listen directives
    server_name: "_"              # Required: Server name(s)
    root: "/var/www/html"         # Optional: Document root
    locations: []                 # Optional: Location blocks
    error_pages: []               # Optional: Custom error pages
    tls: {}                       # Optional: SSL/TLS configuration
```

### Virtual Host Management

Virtual hosts enable hosting multiple websites on a single server, each with its own configuration, SSL certificates, and backend integrations.

**Key Features:**
- Multiple domain hosting
- Independent SSL certificates per domain
- Backend application integration
- Custom logging per virtual host
- Advanced location-based routing
- Upstream server definitions

**Configuration Structure:**
```yaml
nginx_vhost_servers:
  - name: "site_name"             # Required: Unique name
    enabled: true                 # Required: Enable/disable
    server_name: "example.com"    # Required: Domain name(s)
    listen: []                    # Required: Listen directives
    root: "/var/www/site"         # Optional: Document root
    upstreams: []                 # Optional: Backend definitions
    locations: []                 # Optional: Location blocks
    tls: {}                       # Optional: SSL configuration
    access_log: "path"            # Optional: Access log file
    error_log: "path"             # Optional: Error log file
    client_max_body_size: "1m"    # Optional: Upload limit
    error_pages: []               # Optional: Error pages
    extra_config: []              # Optional: Additional directives
```

### HTTP Load Balancing

HTTP load balancers distribute incoming requests across multiple backend servers, providing high availability and scalability for web applications.

**Load Balancing Methods:**
- `round_robin`: Default method, requests distributed evenly
- `least_conn`: Requests sent to server with fewest active connections
- `ip_hash`: Client IP determines backend server (session persistence)
- `hash`: Custom hash key for request distribution

**Health Checking:**
- `max_fails`: Maximum failed attempts before marking server down
- `fail_timeout`: Time to wait before retrying failed server
- `backup`: Server only used when primary servers are unavailable

**Configuration Structure:**
```yaml
nginx_http_loadbalancers:
  - name: "lb_name"               # Required: Unique name
    enabled: true                 # Required: Enable/disable
    lb_method: "least_conn"       # Required: Load balancing method
    listen: []                    # Required: Listen directives
    server_name: "lb.example.com" # Required: Server name
    upstream_servers: []          # Required: Backend servers
    keepalive: 32                 # Optional: Upstream keepalive
    proxy_connect_timeout: "5s"   # Optional: Connection timeout
    proxy_read_timeout: "60s"     # Optional: Read timeout
    proxy_send_timeout: "60s"     # Optional: Send timeout
    client_max_body_size: "10m"   # Optional: Upload limit
    locations: []                 # Optional: Location blocks
    tls: {}                       # Optional: SSL configuration
    error_pages: []               # Optional: Error pages
```

### Stream Load Balancing

Stream load balancers handle TCP and UDP traffic, perfect for database connections, message queues, and other non-HTTP services.

**Use Cases:**
- Database connection pooling
- Message queue load balancing
- TCP service proxying
- SSL termination for non-HTTP services

**Configuration Structure:**
```yaml
nginx_stream_loadbalancers:
  - name: "stream_name"           # Required: Unique name
    enabled: true                 # Required: Enable/disable
    lb_method: "hash"             # Required: Load balancing method
    listen: []                    # Required: Listen directives
    upstream_servers: []          # Required: Backend servers
    proxy_timeout: "60s"          # Optional: Proxy timeout
    proxy_connect_timeout: "5s"   # Optional: Connection timeout
    ssl: false                    # Optional: Enable SSL
    tls: {}                       # Optional: SSL configuration
```

### Static File Serving

Static file servers are optimized for delivering static content like images, CSS, JavaScript, and downloads with maximum performance.

**Optimization Features:**
- Gzip and Brotli compression
- Browser caching headers
- Open file cache for frequently accessed files
- Directory indexing
- Content-type detection
- Range request support

**Configuration Structure:**
```yaml
nginx_static_file_servers:
  - name: "static_name"           # Required: Unique name
    enabled: true                 # Required: Enable/disable
    listen: []                    # Required: Listen directives
    server_name: "static.com"     # Required: Server name
    root: "/var/www/static"       # Required: Content root
    index: "index.html"           # Optional: Index files
    autoindex: false              # Optional: Directory listing
    access_log: "path"            # Optional: Access log
    error_log: "path"             # Optional: Error log
    client_max_body_size: "10M"   # Optional: Upload limit
    locations: []                 # Optional: Location blocks
    tls: {}                       # Optional: SSL configuration
    error_pages: []               # Optional: Error pages
    extra_config: []              # Optional: Additional directives
```

### SSL/TLS Configuration

The role provides comprehensive SSL/TLS configuration with modern security practices and cipher suites.

**Security Features:**
- Modern cipher suites by default
- HSTS (HTTP Strict Transport Security) support
- OCSP stapling configuration
- Perfect Forward Secrecy
- Session resumption optimization

**Best Practices:**
- Use strong cipher suites
- Enable HTTP/2 for performance
- Implement proper certificate chain
- Configure appropriate session timeouts
- Use security headers

### SELinux Integration

When `nginx_selinux_enabled` is true, the role automatically configures SELinux policies for Nginx operation.

**SELinux Features:**
- Automatic policy installation
- Port context configuration
- File context management
- Boolean settings for Nginx features

**Configuration:**
```yaml
nginx_selinux_enabled: true
nginx_selinux_ports: [80, 443, 8080, 8443]
```

### System Optimization

The role includes kernel parameter optimization for high-performance Nginx deployments.

**Optimized Parameters:**
- Network buffer sizes
- Connection limits
- File descriptor limits
- Memory management
- TCP tuning

**Configuration:**
```yaml
nginx_sysctl_parameters:
  - name: net.core.somaxconn
    value: 65535
  - name: net.ipv4.tcp_max_syn_backlog
    value: 8192
  - name: fs.file-max
    value: 2097152
```

## Platform-Specific Considerations

### RedHat Family (CentOS, RHEL, Rocky, AlmaLinux)

- Uses `nginx-all-modules` package for extended functionality
- SELinux integration with `policycoreutils-python-utils`
- Firewalld service management
- SystemD service integration

### Debian Family (Ubuntu, Debian)

- Uses standard `nginx` package
- UFW firewall management
- APT package management
- SystemD service integration

## Security Considerations

1. **SSL/TLS**: Use strong cipher suites and keep certificates updated
2. **SELinux**: Enable SELinux for additional security layers
3. **Firewall**: Configure appropriate firewall rules
4. **Headers**: Implement security headers in virtual hosts
5. **Logging**: Monitor access and error logs for security events
6. **Updates**: Keep Nginx and system packages updated

## Performance Tuning

1. **Workers**: Set `nginx_service_worker_processes` to `auto` or CPU count
2. **Connections**: Increase `nginx_service_worker_connections` for high traffic
3. **File Descriptors**: Set appropriate `nginx_service_worker_rlimit_nofile`
4. **Caching**: Configure appropriate SSL session caching
5. **Compression**: Enable gzip/brotli for static content
6. **Keepalive**: Use upstream keepalive for backend connections

## Troubleshooting

### Common Issues

1. **Permission Errors**: Check SELinux contexts and file permissions
2. **Port Conflicts**: Verify no other services are using configured ports
3. **SSL Issues**: Validate certificate paths and permissions
4. **Backend Connection Failures**: Check upstream server availability
5. **Configuration Syntax**: Use `nginx -t` to test configuration

### Log Analysis

Monitor these log files for issues:
- `/var/log/nginx/error.log`: Main error log
- `/var/log/nginx/*_access.log`: Access logs per service
- `/var/log/nginx/*_error.log`: Error logs per service

### Performance Monitoring

Key metrics to monitor:
- Connection count and rate
- Request rate and response times
- Backend server health
- SSL handshake performance
- Cache hit ratios

## License

BSD-2-Clause

## Author Information

This role has been developed by gkorkmaz

**GitHub**: [https://github.com/ginanck](https://github.com/ginanck)

---

*This role is actively maintained and tested across multiple platforms. For the latest updates and features, please refer to the official repository.*
