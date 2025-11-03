# PHP Apache MySQL/MariaDB Docker Stack

A complete Docker Compose setup for PHP development with Apache, MySQL/MariaDB, and phpMyAdmin.

## Stack Components

- **PHP Apache**: Custom PHP installation with Apache web server
- **MySQL**: Latest MySQL database server
- **MariaDB**: Latest MariaDB database server
- **phpMyAdmin**: Web-based database management interface

## Prerequisites

- Docker Engine 20.10+
- Docker Compose 2.0+
- Git (optional)

## Project Structure

```
.
├── docker-compose.yml
├── .env
├── php-apache/
│   └── Dockerfile
└── src/
    └── (your PHP files here)
```

## Quick Start

### 1. Create Environment File

Create a `.env` file in the project root:

```env
DB_ROOT_PASS=your_root_password
DB_USER=your_database_user
DB_PASS=your_database_password
DB_NAME=your_database_name
```

**Example:**

```env
DB_ROOT_PASS=rootpass123
DB_USER=devuser
DB_PASS=devpass123
DB_NAME=myapp_db
```

### 2. Build and Start Services

```bash
# Build and start all containers
docker compose up -d --build

# View running containers
docker compose ps

# View logs
docker compose logs -f
```

### 3. Stop Services

```bash
# Stop all containers
docker compose down

# Stop and remove volumes (WARNING: deletes all data)
docker compose down -v
```

## Access Points

| Service | URL | Port |
|---------|-----|------|
| PHP Application | <http://localhost:8080> | 8080 |
| phpMyAdmin | <http://localhost:8081> | 8081 |
| MySQL (external) | localhost:33060 | 33060 |
| MariaDB (external) | localhost:33061 | 33061 |

## Database Connection

### From PHP Application (Internal)

**MySQL Connection:**

```php
<?php
$host = 'mysql';
$port = 3306;
$dbname = getenv('DB_NAME');
$user = getenv('DB_USER');
$pass = getenv('DB_PASS');

$dsn = "mysql:host=$host;port=$port;dbname=$dbname";
$pdo = new PDO($dsn, $user, $pass);
?>
```

**MariaDB Connection:**

```php
<?php
$host = 'mariadb';
$port = 3306;
$dbname = getenv('DB_NAME');
$user = getenv('DB_USER');
$pass = getenv('DB_PASS');

$dsn = "mysql:host=$host;port=$port;dbname=$dbname";
$pdo = new PDO($dsn, $user, $pass);
?>
```

### From Host Machine (External)

**MySQL:**

```bash
mysql -h 127.0.0.1 -P 33060 -u root -p
```

**MariaDB:**

```bash
mysql -h 127.0.0.1 -P 33061 -u root -p
```

### Using phpMyAdmin

1. Navigate to <http://localhost:8081>
2. Select server: `mysql` or `mariadb`
3. Login with:
   - Username: `root` or your DB_USER
   - Password: Your DB_ROOT_PASS or DB_PASS

## Volume Management

### Persistent Data Volumes

- `mysql-data`: MySQL database files
- `mariadb-data`: MariaDB database files
- `session-data`: phpMyAdmin session data

### Backup Database

**MySQL:**

```bash
docker exec mysql mysqldump -u root -p${DB_ROOT_PASS} ${DB_NAME} > backup_mysql.sql
```

**MariaDB:**

```bash
docker exec mariadb mysqldump -u root -p${DB_ROOT_PASS} ${DB_NAME} > backup_mariadb.sql
```

### Restore Database

**MySQL:**

```bash
docker exec -i mysql mysql -u root -p${DB_ROOT_PASS} ${DB_NAME} < backup_mysql.sql
```

**MariaDB:**

```bash
docker exec -i mariadb mysql -u root -p${DB_ROOT_PASS} ${DB_NAME} < backup_mariadb.sql
```

## Useful Commands

### Container Management

```bash
# Restart a specific service
docker compose restart php-apache

# View container logs
docker compose logs -f php-apache

# Execute commands in container
docker compose exec php-apache bash
docker compose exec mysql bash
docker compose exec mariadb bash

# Rebuild specific service
docker compose up -d --build php-apache
```

### Database Management

```bash
# Access MySQL CLI
docker compose exec mysql mysql -u root -p

# Access MariaDB CLI
docker compose exec mariadb mysql -u root -p

# Check database status
docker compose exec mysql mysqladmin -u root -p status
```

### Cleanup

```bash
# Remove stopped containers
docker compose rm

# Remove all containers, networks, and volumes
docker compose down -v

# Remove unused Docker resources
docker system prune -a
```

## Troubleshooting

### Port Already in Use

If you get a port conflict error:

```bash
# Check what's using the port
lsof -i :8080
# or
netstat -ano | findstr :8080  # Windows

# Change ports in docker-compose.yml
ports:
  - '8081:80'  # Change 8080 to 8081
```

### Container Won't Start

```bash
# Check container logs
docker compose logs php-apache

# Verify environment variables
docker compose config

# Rebuild from scratch
docker compose down -v
docker compose build --no-cache
docker compose up -d
```

### Permission Issues

```bash
# Fix file permissions (Linux/Mac)
sudo chown -R $USER:$USER src/
chmod -R 755 src/
```

### Database Connection Failed

1. Verify containers are running: `docker compose ps`
2. Check database logs: `docker compose logs mysql`
3. Ensure correct hostname: use `mysql` or `mariadb`, not `localhost`
4. Verify environment variables in `.env` file

## Development Tips

### Hot Reload

The `src` directory is mounted as a volume, so PHP files update automatically without rebuilding.

### Multiple PHP Versions

To use different PHP versions, modify `php-apache/Dockerfile`:

```dockerfile
FROM php:8.2-apache
# or
FROM php:7.4-apache
```

### Installing PHP Extensions

Add to your `php-apache/Dockerfile`:

```dockerfile
RUN docker-php-ext-install mysqli pdo pdo_mysql
```

## Security Notes

⚠️ **This setup is for development only!**

For production:

- Use strong passwords
- Don't expose database ports externally
- Use Docker secrets instead of environment variables
- Enable SSL/TLS
- Restrict phpMyAdmin access
- Keep images updated
