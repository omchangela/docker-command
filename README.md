# docker-command



Here’s a concise list of basic Docker commands tailored for a Laravel application using MySQL, assuming a typical setup with Docker Compose for orchestrating the Laravel app and MySQL database. These commands cover common tasks like building, running, and managing containers.

### Prerequisites
- Docker and Docker Compose installed.
- A `docker-compose.yml` file configured for Laravel (PHP with Nginx/Apache) and MySQL. A basic example:
  ```yaml
  version: '3'
  services:
    app:
      build:
        context: .
        dockerfile: Dockerfile
      image: laravel-app
      ports:
        - "8000:80"
      volumes:
        - .:/var/www/html
      depends_on:
        - db
    db:
      image: mysql:8.0
      environment:
        MYSQL_ROOT_PASSWORD: root
        MYSQL_DATABASE: laravel
        MYSQL_USER: laravel
        MYSQL_PASSWORD: secret
      ports:
        - "3306:3306"
      volumes:
        - db-data:/var/lib/mysql
  volumes:
    db-data:
  ```

- A `Dockerfile` for the Laravel app, e.g.:
  ```Dockerfile
  FROM php:8.1-fpm
  RUN apt-get update && apt-get install -y \
      libpng-dev \
      libjpeg-dev \
      zip \
      unzip \
      && docker-php-ext-install pdo_mysql gd
  RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
  WORKDIR /var/www/html
  COPY . .
  RUN composer install
  CMD php artisan serve --host=0.0.0.0 --port=80
  ```

### Basic Docker Commands for Laravel + MySQL

1. **Build the Docker images**
   ```bash
   docker-compose build
   ```
   Builds the Laravel app image based on the `Dockerfile` and pulls the MySQL image.

2. **Start the containers**
   ```bash
   docker-compose up -d
   ```
   Starts the Laravel app and MySQL containers in detached (background) mode.

3. **Stop the containers**
   ```bash
   docker-compose down
   ```
   Stops and removes the containers, preserving the MySQL data volume.

4. **View running containers**
   ```bash
   docker ps
   ```
   Lists all running containers, including Laravel (`app`) and MySQL (`db`).

5. **Check container logs**
   ```bash
   docker-compose logs app
   ```
   Displays logs for the Laravel app container. Use `db` to check MySQL logs.

6. **Access the Laravel container**
   ```bash
   docker-compose exec app bash
   ```
   Opens a bash shell inside the Laravel app container for running Artisan commands.

7. **Run Laravel Artisan commands**
   ```bash
   docker-compose exec app php artisan <command>
   ```
   Examples:
   - `docker-compose exec app php artisan migrate` – Runs database migrations.
   - `docker-compose exec app php artisan make:model ModelName` – Creates a model.
   - `docker-compose exec app php artisan serve` – Starts the Laravel development server (if not already running).

8. **Access MySQL container**
   ```bash
   docker-compose exec db mysql -u laravel -p
   ```
   Logs into the MySQL container with the user `laravel` (prompts for password, e.g., `secret`).

9. **Check MySQL status**
   ```bash
   docker-compose exec db mysqladmin -u root -p status
   ```
   Shows MySQL server status (prompts for the root password, e.g., `root`).

10. **Rebuild and restart containers**
    ```bash
    docker-compose up -d --build
    ```
    Rebuilds images and restarts containers, useful after code or config changes.

11. **Remove unused containers, images, and volumes**
    ```bash
    docker system prune -a --volumes
    ```
    Cleans up unused Docker resources (use with caution, as it removes all unused images and volumes).

12. **View Docker Compose services**
    ```bash
    docker-compose ps
    ```
    Lists the status of services defined in `docker-compose.yml`.

### Laravel-Specific Tips
- **Environment Configuration**: Ensure the Laravel `.env` file has correct MySQL settings:
  ```env
  DB_CONNECTION=mysql
  DB_HOST=db
  DB_PORT=3306
  DB_DATABASE=laravel
  DB_USERNAME=laravel
  DB_PASSWORD=secret
  ```
- **Composer Commands**: Run Composer inside the Laravel container:
  ```bash
  docker-compose exec app composer install
  ```
- **Clear Cache**: Clear Laravel caches if needed:
  ```bash
  docker-compose exec app php artisan cache:clear
  docker-compose exec app php artisan config:cache
  ```

### MySQL-Specific Tips
- **Persistent Data**: The `db-data` volume ensures MySQL data persists across container restarts.
- **Backup Database**:
  ```bash
  docker-compose exec db mysqldump -u laravel -p laravel > backup.sql
  ```
  Exports the `laravel` database to `backup.sql` (prompts for password).
- **Restore Database**:
  ```bash
  docker-compose exec db mysql -u laravel -p laravel < backup.sql
  ```

### Notes
- Replace `app` and `db` with your service names if different in `docker-compose.yml`.
- Ensure ports `8000` (Laravel) and `3306` (MySQL) are free or adjust in the `docker-compose.yml`.
- For production, configure a web server like Nginx and optimize the PHP setup.

If you need help with a specific command or setup, let me know!
