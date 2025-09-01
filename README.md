# Dev Stack Setup

This Docker Compose setup provides a complete development stack with MySQL, Elasticsearch, Memcached, and Redis.

## Features

- **MySQL 8.0** server with 8 pre-configured databases
- **Elasticsearch 8.11.0** for search and analytics
- **Memcached 1.6** for high-performance caching
- **Redis 7** for advanced caching and data structures
- **Custom network** (`dev-stack-network`) for container communication
- **Persistent data** storage using Docker volumes
- **Automatic database import** from the `mysql-init/` folder

## Services

### MySQL 8.0
- **SQL Mode:** Disabled (`sql_mode = ""`) for maximum compatibility - customize as needed
- **Authentication:** `mysql_native_password` - customize as needed

### Elasticsearch 8.11.0
- Single-node setup for development
- Security disabled for easy access
- 512MB heap size allocated

### Memcached 1.6
- 256MB memory limit
- High-performance caching

### Redis 7
- Persistence enabled with AOF
- Advanced data structures support

## Quick Start

1. **Start all services:**
   ```bash
   docker-compose up -d
   ```

2. **Check if all containers are running:**
   ```bash
   docker-compose ps
   ```

3. **View logs for specific service:**
   ```bash
   docker-compose logs mysql
   docker-compose logs elasticsearch
   docker-compose logs memcached
   docker-compose logs redis
   ```

4. **Connect to services:**
   ```bash
   # MySQL
   mysql -h localhost -P 3306 -u root -p
   # Password: rootpassword
   
   mysql -h localhost -P 3306 -u admin -p
   # Password: admin123
   
   # Elasticsearch
   curl http://localhost:9200
   
   # Redis
   redis-cli -h localhost -p 6379
   
   # Memcached
   telnet localhost 11211
   ```

## Connection Details

### MySQL
**Root User:**
- **Host:** `localhost` (from host machine) or `mysql` (from other containers)
- **Port:** `3306`
- **Username:** `root`
- **Password:** `rootpassword`
- **Network:** `dev-stack-network`

**Custom User (example):**
- **Host:** `localhost` (from host machine) or `mysql` (from other containers)
- **Port:** `3306`
- **Username:** `admin` (example - customize as needed)
- **Password:** `admin123` (example - customize as needed)
- **Authentication:** `mysql_native_password`
- **Network:** `dev-stack-network`
- **Privileges:** Full access to all databases
- **Note:** Create by copying `99_init-user.sql.example` to `99_init-user.sql`

### Elasticsearch
- **Host:** `localhost` (from host machine) or `elasticsearch` (from other containers)
- **Port:** `9200` (HTTP), `9300` (Transport)
- **URL:** `http://localhost:9200`
- **Network:** `dev-stack-network`
- **Security:** Disabled for development

### Redis
- **Host:** `localhost` (from host machine) or `redis` (from other containers)
- **Port:** `6379`
- **Network:** `dev-stack-network`
- **Persistence:** AOF enabled

### Memcached
- **Host:** `localhost` (from host machine) or `memcached` (from other containers)
- **Port:** `11211`
- **Memory:** 256MB
- **Network:** `dev-stack-network`

## Database Import

The databases are automatically imported when the container starts for the first time. The import process:

1. MySQL server starts
2. SQL files from `mysql-init/` folder are executed in alphabetical order
3. Each file creates its respective database and imports the data

### User Setup

To create custom users, copy the example file and modify it:

```bash
# Copy the example user creation script
cp mysql-init/99_init-user.sql.example mysql-init/99_init-user.sql

# Edit the file to customize your users
nano mysql-init/99_init-user.sql
```

The example file contains a template for creating the `admin` user. Modify it to create your own users with custom usernames, passwords, and privileges.

## Management Commands

```bash
# Stop all services
docker-compose down

# Stop and remove volumes (WARNING: This will delete all data)
docker-compose down -v

# Restart all services
docker-compose restart

# Restart specific service
docker-compose restart mysql
docker-compose restart elasticsearch
docker-compose restart redis
docker-compose restart memcached

# View real-time logs
docker-compose logs -f mysql
docker-compose logs -f elasticsearch
docker-compose logs -f redis
docker-compose logs -f memcached

# Access service shells
docker-compose exec mysql mysql -u root -p
docker-compose exec redis redis-cli
docker-compose exec elasticsearch bash

# MySQL specific commands
docker-compose exec mysql mysqldump -u root -p database_name > backup.sql
docker-compose exec -T mysql mysql -u root -p database_name < backup.sql

# Elasticsearch health check
curl http://localhost:9200/_cluster/health

# Redis info
docker-compose exec redis redis-cli info
```

## Network Usage

Other containers can connect to all services using the service names as hostnames:
- **MySQL:** `mysql:3306`
- **Elasticsearch:** `elasticsearch:9200`
- **Redis:** `redis:6379`
- **Memcached:** `memcached:11211`
- **Network:** `dev-stack-network`

Example for another service in the same network:
```yaml
services:
  my-app:
    image: my-app
    networks:
      - dev-stack-network
    environment:
      # MySQL
      - DB_HOST=mysql
      - DB_PORT=3306
      - DB_USER=admin
      - DB_PASSWORD=admin123
      
      # Elasticsearch
      - ES_HOST=elasticsearch
      - ES_PORT=9200
      
      # Redis
      - REDIS_HOST=redis
      - REDIS_PORT=6379
      
      # Memcached
      - MEMCACHED_HOST=memcached
      - MEMCACHED_PORT=11211
```

## Troubleshooting

1. **Port already in use:** Change the port mappings in `docker-compose.yml`:
   - MySQL: `"3307:3306"`
   - Elasticsearch: `"9201:9200"`
   - Redis: `"6380:6379"`
   - Memcached: `"11212:11211"`

2. **MySQL import taking too long:** Large database files may take several minutes to import. Check logs with `docker-compose logs -f mysql`

3. **SQL Mode:** MySQL is configured with `sql_mode = ""` (disabled) for maximum compatibility with legacy applications, You can change it if you want

4. **Elasticsearch memory issues:** If Elasticsearch fails to start, increase memory allocation by changing `ES_JAVA_OPTS` in docker-compose.yml

5. **Permission issues:** Ensure the `mysql-init/` folder has proper read permissions

6. **Service health checks:**
   ```bash
   # Check all services
   docker-compose ps
   
   # Check Elasticsearch health
   curl http://localhost:9200/_cluster/health
   
   # Check Redis
   docker-compose exec redis redis-cli ping
   
   # Check MySQL
   docker-compose exec mysql mysqladmin ping
   ```

7. **Reset everything:** 
   ```bash
   docker-compose down -v
   docker-compose up -d
   ```
