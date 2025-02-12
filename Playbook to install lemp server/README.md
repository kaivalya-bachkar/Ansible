A LEMP stack consists of Linux, Nginx, MySQL (or MariaDB), and PHP. Below is an Ansible playbook to install and configure the LEMP stack on a server running a Linux distribution such as Ubuntu.

```yaml
---
- name: Install LEMP Stack
  hosts: all
  become: yes

  tasks:
    - name: Update apt packages
      apt:
        update_cache: yes
        force_apt_get: yes

    - name: Install Nginx
      apt:
        name: nginx
        state: present

    - name: Start and enable Nginx service
      systemd:
        name: nginx
        enabled: yes
        state: started

    - name: Install MySQL server
      apt:
        name: mysql-server
        state: present

    - name: Install MySQL client
      apt:
        name: mysql-client
        state: present

    - name: Start and enable MySQL service
      systemd:
        name: mysql
        enabled: yes
        state: started

    - name: Install PHP and related modules
      apt:
        name:
          - php
          - php-fpm
          - php-mysql
        state: present

    - name: Configure Nginx to use PHP Processor
      blockinfile:
        path: /etc/nginx/sites-available/default
        block: |
          server {
              listen 80 default_server;
              listen [::]:80 default_server;

              root /var/www/html;
              index index.php index.html index.htm index.nginx-debian.html;

              server_name _;

              location / {
                  try_files $uri $uri/ =404;
              }

              location ~ \.php$ {
                  include snippets/fastcgi-php.conf;
                  fastcgi_pass unix:/var/run/php/php-fpm.sock;
              }

              location ~ /\.ht {
                  deny all;
              }
          }

    - name: Test Nginx configuration
      command: nginx -t

    - name: Reload Nginx
      systemd:
        name: nginx
        state: reloaded

    - name: Create a PHP info file
      copy:
        dest: /var/www/html/info.php
        content: |
          <?php
          phpinfo();
          ?>

    - name: Set proper permissions on web root
      file:
        path: /var/www/html
        owner: www-data
        group: www-data
        mode: '0755'
        state: directory
```

### Playbook Breakdown:

1. **Updating apt packages:** Updates the package index.
2. **Installing Nginx:** Installs Nginx and ensures it is running and enabled.
3. **Installing MySQL:** Installs MySQL server and client.
4. **Installing PHP:** Installs PHP and the necessary PHP modules for Nginx to work with PHP.
5. **Nginx Configuration:** Modifies the default Nginx configuration to process PHP files.
6. **Reloading Nginx:** Reloads Nginx after configuration changes.
7. **Creating a PHP info file:** Adds a sample `info.php` file to verify PHP is working.
8. **Setting permissions:** Ensures the web root has the correct permissions.

This playbook assumes Ubuntu as the target system. For different distributions (e.g., CentOS), the package manager and service names will need adjustments.
