
## Self-Hosted Web Server and Web Applications

This section covers the installation and configuration of several popular web services on Ubuntu Server. These include web servers (Apache and Nginx), databases (MariaDB and MongoDB), and various web applications (WordPress, Observium, phpMyAdmin, Rocket.Chat). Where applicable, Docker Compose is used for simplified and reproducible deployments.

---

### Apache and Nginx

#### Apache

```bash
sudo apt update
sudo apt install apache2
```

- Enable and start the service:

```bash
sudo systemctl enable apache2
sudo systemctl start apache2
```

- Allow HTTP/HTTPS through the firewall:

```bash
sudo ufw allow "Apache Full"
```

- Default document root: `/var/www/html`
- Configuration files: `/etc/apache2/sites-available/`
- Enable a site:

```bash
sudo a2ensite your-site.conf
sudo systemctl reload apache2
```

#### Nginx

```bash
sudo apt update
sudo apt install nginx
```

- Enable and start the service:

```bash
sudo systemctl enable nginx
sudo systemctl start nginx
```

- Allow HTTP/HTTPS through the firewall:

```bash
sudo ufw allow "Nginx Full"
```

- Default document root: `/var/www/html`
- Configuration files: `/etc/nginx/sites-available/`
- Enable a site:

```bash
sudo ln -s /etc/nginx/sites-available/your-site /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

---

### MariaDB and MongoDB

#### MariaDB

```bash
sudo apt update
sudo apt install mariadb-server
```

- Secure installation:

```bash
sudo mysql_secure_installation
```

- Enable and start the service:

```bash
sudo systemctl enable mariadb
sudo systemctl start mariadb
```

- Log into MariaDB:

```bash
sudo mariadb -u root -p
```

#### MongoDB

```bash
sudo apt update
sudo apt install mongodb
```

- Enable and start the service:

```bash
sudo systemctl enable mongodb
sudo systemctl start mongodb
```

- Check the status:

```bash
sudo systemctl status mongodb
```

---

### Web Applications

#### WordPress (Apache + MariaDB)

- Install dependencies:

```bash
sudo apt install php php-mysql libapache2-mod-php php-cli php-cgi php-gd
```

- Download WordPress:

```bash
cd /tmp
curl -O https://wordpress.org/latest.tar.gz
tar xzvf latest.tar.gz
sudo mv wordpress /var/www/html/
```

- Set permissions:

```bash
sudo chown -R www-data:www-data /var/www/html/wordpress
sudo chmod -R 755 /var/www/html/wordpress
```

- Create a MariaDB database and user for WordPress:

```sql
CREATE DATABASE wordpress;
CREATE USER 'wpuser'@'localhost' IDENTIFIED BY 'your_password';
GRANT ALL PRIVILEGES ON wordpress.* TO 'wpuser'@'localhost';
FLUSH PRIVILEGES;
```

- Complete installation via web browser at `http://your_server_ip/wordpress`

---

#### Observium (Apache + MySQL)

- Install dependencies:

```bash
sudo apt install apache2 mariadb-server php php-mysql php-snmp rrdtool snmp snmpd fping git
```

- Create Observium user and database:

```sql
CREATE DATABASE observium DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;
GRANT ALL PRIVILEGES ON observium.* TO 'observium'@'localhost' IDENTIFIED BY 'your_password';
FLUSH PRIVILEGES;
```

- Download and install Observium:

```bash
cd /opt
sudo git clone https://github.com/observium/observium.git
cd observium
sudo ./discovery.php -u
```

- Add a device and start discovery:

```bash
sudo ./add_device.php your-hostname public v2c
sudo ./discovery.php -h all
sudo ./poller.php -h all
```

---

#### phpMyAdmin (Apache + MariaDB)

```bash
sudo apt install phpmyadmin
```

- Choose Apache2 when prompted.
- Configure database for phpMyAdmin.
- Enable the phpMyAdmin configuration:

```bash
sudo ln -s /etc/phpmyadmin/apache.conf /etc/apache2/conf-available/phpmyadmin.conf
sudo a2enconf phpmyadmin
sudo systemctl reload apache2
```

- Access via: `http://your_server_ip/phpmyadmin`

---

#### Rocket.Chat (Docker Compose)

- Create a `docker-compose.yml`:

```yaml
version: "3"

services:
  rocketchat:
    image: registry.rocket.chat/rocketchat/rocket.chat:latest
    restart: unless-stopped
    environment:
      - MONGO_URL=mongodb://mongo:27017/rocketchat
      - ROOT_URL=http://localhost:3000
      - PORT=3000
    ports:
      - 3000:3000
    depends_on:
      - mongo

  mongo:
    image: mongo:5.0
    restart: unless-stopped
    volumes:
      - ./data/db:/data/db
```

- Run it:

```bash
docker compose up -d
```

- Access via: `http://your_server_ip:3000`

---

Let me know if you'd like to add Nextcloud, MediaWiki, Mattermost, or any other services next.