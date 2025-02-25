# Custom MySQL and PHPMyAdmin Database Setup for WordPress

This guide provides step-by-step instructions to set up a custom MySQL database and PHPMyAdmin for WordPress using Docker. The setup includes MySQL, PHPMyAdmin, and WordPress configurations, as well as how to import an SQL file into the MySQL database.

## Prerequisites

Before proceeding, ensure you have the following:

- **Docker** installed on your machine.
- **Docker Compose** installed on your machine.
- A **SQL file** (e.g., `wp_stgmfs5.sql.gz`) that you want to import into the MySQL database.

## Step 1: Docker Setup

### 1.1 Create a `docker-compose.yml` File

Create a directory for your project, and inside it, create a `docker-compose.yml` file. This file will define your MySQL, PHPMyAdmin, and WordPress containers.

```yaml
version: '3.8'

services:
  db:
    image: mysql:8.0.41-debian
    container_name: mysql-container
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: local
    ports:
      - "3307:3306"
    volumes:
      - db_data:/var/lib/mysql

  phpmyadmin:
    image: phpmyadmin/phpmyadmin
    container_name: phpmyadmin-container
    depends_on:
      - db
    ports:
      - "8080:80"
    environment:
      PMA_HOST: db
      PMA_USER: root
      PMA_PASSWORD: root
    volumes:
      - ./custom.ini:/usr/local/etc/php/conf.d/custom.ini

volumes:
  db_data:
```

### 1.2 Start the Docker Containers

To start the MySQL and PHPMyAdmin containers, run the following command in the same directory as your `docker-compose.yml` file:

```bash
docker-compose up -d
```

This will start the containers in the background. Your MySQL container will be available on port `3307` and PHPMyAdmin will be available on port `8080`.

### 1.3 Access PHPMyAdmin

After the containers are running, you can access PHPMyAdmin through your browser:

```
http://localhost:8080
```

- **Username**: root
- **Password**: root
- **Server**: db (This is the MySQL service name in the `docker-compose.yml` file)


## Step 2: Prepare the SQL File

### 2.1 Copy the SQL File into the Container

Once you have your SQL file (e.g., `wp_stgmfs5.sql.gz`), copy it into the MySQL container using the following command:

```bash
docker cp wp_stgmfs5.sql.gz mysql-container:/wp_stgmfs5.sql.gz
```

### 2.2 Decompress the SQL File

Before importing the `.gz` file, you need to decompress it. Use `gunzip` to decompress the file inside the MySQL container. First, access the container:

```bash
docker exec -it mysql-container bash
```

Then, decompress the file:

```bash
gunzip /wp_stgmfs5.sql.gz
```

This will decompress the file and remove the `.gz` extension, leaving the SQL file as `wp_stgmfs5.sql`.

### 2.3 Install `pv` Package in the MySQL Container

Since your SQL file may be large, we can use the `pv` (Pipe Viewer) tool to monitor the import process. First, update the package list and install `pv`:

```bash
apt update && apt install -y pv
```

If `apt` doesn't work, you can try using `microdnf` instead:

```bash
microdnf install pv
```

### 2.4 Import the SQL File

After installing `pv` and decompressing the SQL file, use the following command to import the SQL file into the MySQL database:

```bash
pv /wp_stgmfs5.sql | mysql -u root -proot local
```

This command will:

- Use `pv` to provide a progress bar as the SQL file is imported.
- Pipe the contents of `wp_stgmfs5.sql` into the MySQL database `local`.

---

With this update, the SQL file is now properly decompressed before being imported into MySQL.




## Step 3: Configure WordPress to Connect to the Database

### 3.1 Update the `wp-config.php` File

In your WordPress installation, edit the `wp-config.php` file to configure the database connection settings. Update the following constants:

```php
define( 'DB_NAME', 'local' );
define( 'DB_USER', 'root' );
define( 'DB_PASSWORD', 'root' );
define( 'DB_HOST', '0.0.0.0:3307' );  // Ensure this points to the correct MySQL port
define( 'DB_HOST_SLAVE', '0.0.0.0:3307' );
define('DB_CHARSET', 'utf8');
define('DB_COLLATE', 'utf8_unicode_ci');
```

This ensures that WordPress connects to the MySQL database in the Docker container.

## Step 4: Verifying the Setup

### 4.1 Verify Database Import

You can verify that the database was successfully imported by logging into PHPMyAdmin (`http://localhost:8080`), using the credentials:

- **Username**: root
- **Password**: root

Navigate to the `local` database and check if the tables from the SQL file have been imported correctly.

### 4.2 Access Your WordPress Site

Once WordPress is connected to the MySQL database, you can access your WordPress site in the browser. The site should be using the database that was imported earlier.

## Step 5: Stopping and Restarting the Containers

### 5.1 Stop the Containers

To stop the containers, use the following command:

```bash
docker-compose down
```

This will stop and remove the containers, but the database data will persist in the `db_data` volume.

### 5.2 Restart the Containers

To restart the containers, run:

```bash
docker-compose up -d
```

This will start the containers again in detached mode.

## Troubleshooting

### Common Issues

- **MySQL Container Not Starting**: If you encounter issues with the MySQL container not starting, try removing the existing volume and recreating it. For example:

```bash
docker-compose down -v
docker-compose up -d
```

- **MySQL Import Error**: If the SQL file import fails, ensure that the file is correctly formatted and doesn't have errors. Check the MySQL error logs inside the container for more information:

```bash
docker logs mysql-container
```

---

## Conclusion

You now have a working setup with MySQL, PHPMyAdmin, and WordPress running in Docker containers. You’ve also learned how to import large SQL files into MySQL using `pv` for monitoring. 

For future reference, you can use this `README.md` to quickly set up a similar environment again.
```

This complete `README.md` file includes all the commands, steps, and troubleshooting tips to ensure users can easily set up a custom MySQL and PHPMyAdmin database for WordPress using Docker.
