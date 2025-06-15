# Multi-PHP Development Environment with Nginx, MySQL, PostgreSQL, Redis

This project provides a Docker Compose-based local development environment designed to support multiple PHP versions (7.0, 7.4, 8.3) working alongside Nginx, MySQL, PostgreSQL, and Redis services. It allows you to simultaneously run and test projects requiring different PHP versions on the same host machine.

## Directory Structure

```
.
├── .env                        # Environment variables configuration file (sensitive info and defaults)
├── .gitignore                  # Git ignore file settings, includes .env
├── docker-compose.yml          # Main Docker Compose configuration file
├── app/                        # Your project code will be placed here
│   ├── php70/                  # PHP 7.0 project directory
│   │   └── index.php           # Test PHP 7.0
│   ├── php74/                  # PHP 7.4 project directory
│   │   └── index.php           # Test PHP 7.4
│   └── php83/                  # PHP 8.3 project directory
│       └── index.php           # Test PHP 8.3
├── nginx/                      # Nginx service related files
│   ├── Dockerfile              # Nginx Dockerfile
│   └── conf.d/                 # Nginx website configuration files
│       ├── default.conf        # Default website (PHP 8.3)
│       ├── php-7.0-website.conf # PHP 7.0 website configuration
│       └── php-7.4-website.conf # PHP 7.4 website configuration
├── php-fpm-7.0/                # PHP 7.0 FPM service related files
│   ├── Dockerfile              # PHP 7.0 FPM Dockerfile
│   ├── php.ini                 # Custom PHP 7.0 INI settings
│   └── www.conf                # Custom PHP 7.0 FPM Pool settings
├── php-fpm-7.4/                # PHP 7.4 FPM service related files
│   ├── Dockerfile              # PHP 7.4 FPM Dockerfile
│   ├── php.ini                 # Custom PHP 7.4 INI settings
│   └── www.conf                # Custom PHP 7.4 FPM Pool settings
├── php-fpm-8.3/                # PHP 8.3 FPM service related files
│   ├── Dockerfile              # PHP 8.3 FPM Dockerfile
│   ├── php.ini                 # Custom PHP 8.3 INI settings
│   └── www.conf                # Custom PHP 8.3 FPM Pool settings
├── mysql/                      # MySQL service related files
│   └── custom.cnf              # Custom MySQL settings (optional)
└── postgres/                   # PostgreSQL service related files
    └── postgresql.conf         # Custom PostgreSQL settings
```

## Services Overview

| Service Name   | Image             | Exposed Ports (Host:Container) | Internal Service Name (for inter-container) | Description                                  |
| :------------- | :---------------- | :----------------------------- | :------------------------------------------ | :------------------------------------------- |
| `php-fpm-7.0`  | Custom (PHP 7.0)  | `N/A` (Exposed: 9000)          | `php-fpm-7.0:9000`                          | PHP 7.0 FastCGI Process Manager              |
| `php-fpm-7.4`  | Custom (PHP 7.4)  | `N/A` (Exposed: 9000)          | `php-fpm-7.4:9000`                          | PHP 7.4 FastCGI Process Manager              |
| `php-fpm-8.3`  | Custom (PHP 8.3)  | `N/A` (Exposed: 9000)          | `php-fpm-8.3:9000`                          | PHP 8.3 FastCGI Process Manager              |
| `nginx`        | `nginx:alpine`    | `80:80`, `443:443`             | `N/A`                                       | Web server, reverse proxy to PHP-FPM         |
| `mysql`        | `mysql:8.0`       | `3306:3306`                    | `mysql:3306`                                | MySQL database service                       |
| `postgres`     | `postgres:15`     | `5432:5432`                    | `postgres:5432`                             | PostgreSQL database service                  |
| `redis`        | `redis:latest`    | `6379:6379`                    | `redis:6379`                                | Redis caching service                        |

## Environment Variables (.env)

This project uses a `.env` file to manage sensitive information and configurable environment variables. When running `docker-compose up`, it automatically reads the `.env` file in the same directory.

Please create a file named `.env` in the same directory as your `docker-compose.yml` file, and fill it with the following content (**be sure to replace with your own strong passwords**):

```dotenv
# .env file
MYSQL_ROOT_PASSWORD=your_mysql_root_password
MYSQL_DATABASE=your_mysql_database
MYSQL_USER=your_mysql_user
MYSQL_PASSWORD=your_mysql_password

POSTGRES_DB=your_pg_database
POSTGRES_USER=your_pg_user
POSTGRES_PASSWORD=your_pg_password

# REDIS_PASSWORD=your_redis_password # Uncomment and set if your Redis requires a password
```

If variables in the `.env` file are not defined, `docker-compose.yml` will use default values (e.g., `MYSQL_ROOT_PASSWORD` will default to `root_password` if not set in `.env` or as a system environment variable).

## How to Get Started

1.  **Clone the Project:**
    ```bash
    git clone your-repo-url
    cd multi-php-docker
    ```
2.  **Create `app` directories and test files:**
    Ensure that `app/php70`, `app/php74`, `app/php83` directories exist, and place an `index.php` file with `<?php phpinfo(); ?>` in each for testing.
    *Example: `app/php70/index.php`*
    ```php
    <?php phpinfo(); ?>
    ```
3.  **Create `.env` file:**
    Create a `.env` file in the same directory as your `docker-compose.yml` and populate it with your environment variables.
4.  **Configure Host `hosts` file:**
    Edit your host machine's `hosts` file (Windows: `C:\Windows\System32\drivers\etc\hosts` | Linux/macOS: `/etc/hosts`) and add the following lines to map domains to your Docker network:
    ```
    127.0.0.1 php7-0.localhost
    127.0.0.1 php7-4.localhost
    127.0.0.1 php8-3.localhost
    127.0.0.1 localhost # If you use default.conf for localhost access
    ```
5.  **Start Docker Compose services:**
    ```bash
    docker-compose up --build -d
    ```
    * `--build`: Builds (or rebuilds) images.
    * `-d`: Runs containers in the background.

## Website Access

Once successfully started, you should be able to access different PHP versions via the following URLs:

* **PHP 7.0 Project:** `http://php7-0.localhost/`
* **PHP 7.4 Project:** `http://php7-4.localhost/`
* **PHP 8.3 Project (default site):** `http://php8-3.localhost/` or `http://localhost/` (depending on the `server_name` configured in `nginx/conf.d/default.conf`)

## Database and Cache Connection Information (within PHP applications)

Your PHP application containers (`php-fpm-7.0`, `php-fpm-7.4`, `php-fpm-8.3`) should use the following service names and ports to connect to your databases and Redis:

| Service      | Host        | Port   | Username           | Password           | Database Name        |
| :----------- | :---------- | :----- | :----------------- | :----------------- | :------------------- |
| **MySQL** | `mysql`     | `3306` | `${MYSQL_USER}`    | `${MYSQL_PASSWORD}`| `${MYSQL_DATABASE}`  |
| **PostgreSQL** | `postgres`  | `5432` | `${POSTGRES_USER}` | `${POSTGRES_PASSWORD}`| `${POSTGRES_DB}`     |
| **Redis** | `redis`     | `6379` | `N/A`              | `${REDIS_PASSWORD}` (if set) | `N/A`                |

**Example (PHP connecting to MySQL):**

```php
<?php
$dsn = 'mysql:host=mysql;port=3306;dbname=' . getenv('MYSQL_DATABASE') . ';charset=utf8mb4';
$username = getenv('MYSQL_USER');
$password = getenv('MYSQL_PASSWORD');

try {
    $pdo = new PDO($dsn, $username, $password);
    echo "Connected to MySQL successfully!";
} catch (PDOException $e) {
    echo "Connection failed: " . $e->getMessage();
}
```

## Common Issues and Troubleshooting

* **`502 Bad Gateway`**:
    * Check `docker ps` to confirm all PHP-FPM and Nginx containers are running.
    * Check `docker logs nginx` and `docker logs php-fpm-<version>` for error messages.
    * Verify that `fastcgi_pass` in your Nginx configurations (`nginx/conf.d/*.conf`) points to the correct PHP-FPM service name and port (should be `php-fpm-X.X:9000`).
    * Confirm that `listen = 0.0.0.0:9000` or `listen = 9000` is set in the `www.conf` file for each PHP-FPM version (e.g., `php-fpm-8.3/www.conf`), ensuring it listens on port 9000. The provided `www.conf` samples currently use `127.0.0.1:9000`, which might cause issues for inter-container communication; changing it to `0.0.0.0:9000` or simply `9000` is recommended.
* **Missing PHP Extensions:**
    * If your application requires specific extensions, edit the corresponding PHP version's `Dockerfile` and add them within the `RUN apt-get install -y` and `docker-php-ext-install` lines.
    * For PECL extensions (e.g., `imagick`, `redis`, `amqp`), ensure they are installed via `pecl install` and then enabled by creating an `.ini` file like `echo "extension=extension_name.so" > /usr/local/etc/php/conf.d/20-extension_name.ini`.
* **Database Connection Issues:**
    * Check `docker logs mysql` / `docker logs postgres` / `docker logs redis` for any startup errors in the database/cache services.
    * Ensure your PHP application uses the correct service names (`mysql`, `postgres`, `redis`) and ports.
    * Verify that the `environment` variables for your database services in `docker-compose.yml` match the credentials used in your application.
* **Resource Exhaustion:**
    * If your machine has limited RAM or CPU, running multiple services might lead to performance issues. Consider increasing Docker Desktop's resource limits or stopping unnecessary services.

## Cleaning Up the Environment

To stop and remove all services, networks, and volumes (which will erase all database data), run:

```bash
docker-compose down --volumes
```
