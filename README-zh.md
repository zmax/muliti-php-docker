# Multi-PHP Development Environment with Nginx, MySQL, PostgreSQL, Redis

這個專案提供了一個基於 Docker Compose 的本地開發環境，旨在支援多個 PHP 版本（7.0, 7.4, 8.3）與 Nginx、MySQL、PostgreSQL 和 Redis 服務的協同工作。它允許您在同一宿主機上同時運行和測試需要不同 PHP 版本的專案。

## 目錄結構

```
.
├── .env                        # 環境變數設定檔 (敏感資訊和預設值)
├── .gitignore                  # Git 忽略檔案設定，包含 .env
├── docker-compose.yml          # Docker Compose 主設定檔
├── app/                        # 您的專案程式碼將放在這裡
│   ├── php70/                  # PHP 7.0 專案目錄
│   │   └── index.php           # 測試 PHP 7.0
│   ├── php74/                  # PHP 7.4 專案目錄
│   │   └── index.php           # 測試 PHP 7.4
│   └── php83/                  # PHP 8.3 專案目錄
│       └── index.php           # 測試 PHP 8.3
├── nginx/                      # Nginx 服務相關檔案
│   ├── Dockerfile              # Nginx Dockerfile
│   └── conf.d/                 # Nginx 網站設定檔
│       ├── default.conf        # 預設網站 (PHP 8.3)
│       ├── php-7.0-website.conf # PHP 7.0 網站配置
│       └── php-7.4-website.conf # PHP 7.4 網站配置
├── php-fpm-7.0/                # PHP 7.0 FPM 服務相關檔案
│   ├── Dockerfile              # PHP 7.0 FPM Dockerfile
│   ├── php.ini                 # 自定義 PHP 7.0 INI 設定
│   └── www.conf                # 自定義 PHP 7.0 FPM Pool 設定
├── php-fpm-7.4/                # PHP 7.4 FPM 服務相關檔案
│   ├── Dockerfile              # PHP 7.4 FPM Dockerfile
│   ├── php.ini                 # 自定義 PHP 7.4 INI 設定
│   └── www.conf                # 自定義 PHP 7.4 FPM Pool 設定
├── php-fpm-8.3/                # PHP 8.3 FPM 服務相關檔案
│   ├── Dockerfile              # PHP 8.3 FPM Dockerfile
│   ├── php.ini                 # 自定義 PHP 8.3 INI 設定
│   └── www.conf                # 自定義 PHP 8.3 FPM Pool 設定
├── mysql/                      # MySQL 服務相關檔案
│   └── custom.cnf              # 自定義 MySQL 設定 (可選)
└── postgres/                   # PostgreSQL 服務相關檔案
    └── postgresql.conf         # 自定義 PostgreSQL 設定
```

## 服務概覽

| 服務名稱     | 映像檔            | 對外端口 (宿主機:容器) | 內部服務名稱 (容器內連接) | 說明                                      |
| :----------- | :---------------- | :--------------------- | :------------------------ | :---------------------------------------- |
| `php-fpm-7.0` | 自定義 (PHP 7.0)  | `N/A` (Exposed: 9000)  | `php-fpm-7.0:9000`        | PHP 7.0 FastCGI 進程管理器                 |
| `php-fpm-7.4` | 自定義 (PHP 7.4)  | `N/A` (Exposed: 9000)  | `php-fpm-7.4:9000`        | PHP 7.4 FastCGI 進程管理器                 |
| `php-fpm-8.3` | 自定義 (PHP 8.3)  | `N/A` (Exposed: 9000)  | `php-fpm-8.3:9000`        | PHP 8.3 FastCGI 進程管理器                 |
| `nginx`      | `nginx:alpine`    | `80:80`, `443:443`     | `N/A`                     | 網頁伺服器，反向代理到 PHP-FPM            |
| `mysql`      | `mysql:8.0`       | `3306:3306`            | `mysql:3306`              | MySQL 資料庫服務                           |
| `postgres`   | `postgres:15`     | `5432:5432`            | `postgres:5432`           | PostgreSQL 資料庫服務                      |
| `redis`      | `redis:latest`    | `6379:6379`            | `redis:6379`              | Redis 快取服務                             |

## 環境變數 (.env)

這個專案使用 `.env` 檔案來管理敏感資訊和可配置的環境變數。 在運行 `docker-compose up` 時，它會自動讀取同目錄下的 `.env` 檔案。

請在 `docker-compose.yml` 所在的目錄下創建一個名為 `.env` 的檔案，並填入以下內容（**請務必替換為您自己的強密碼**）：

```dotenv
# .env file
MYSQL_ROOT_PASSWORD=your_mysql_root_password
MYSQL_DATABASE=your_mysql_database
MYSQL_USER=your_mysql_user
MYSQL_PASSWORD=your_mysql_password

POSTGRES_DB=your_pg_database
POSTGRES_USER=your_pg_user
POSTGRES_PASSWORD=your_pg_password

# REDIS_PASSWORD=your_redis_password # 如果你的 Redis 需要密碼，請取消註解並設定
```

如果 `.env` 檔案中的變數未定義，`docker-compose.yml` 將使用預設值（例如 `MYSQL_ROOT_PASSWORD` 會預設為 `root_password`）。

## 如何啟動

1.  **複製專案：**
    ```bash
    git clone your-repo-url
    cd multi-php-docker
    ```
2.  **創建 `app` 目錄和測試檔案：**
    確保 `app/php70`, `app/php74`, `app/php83` 目錄存在，並在每個目錄中放入一個 `index.php` 檔案，內容為 `<?php phpinfo(); ?>` 以便測試。
    *範例：`app/php70/index.php`*
    ```php
    <?php phpinfo(); ?>
    ```
3.  **創建 `.env` 檔案：**
    在 `docker-compose.yml` 檔案的同級目錄下創建 `.env` 檔案，並填寫您的環境變數。
4.  **配置宿主機 `hosts` 檔案：**
    編輯您的宿主機 `hosts` 檔案（Windows: `C:\Windows\System32\drivers\etc\hosts` | Linux/macOS: `/etc/hosts`），添加以下行以將域名映射到 Docker 網路：
    ```
    127.0.0.1 php7-0.localhost
    127.0.0.1 php7-4.localhost
    127.0.0.1 php8-3.localhost
    127.0.0.1 localhost # 如果您使用 default.conf 作為 localhost 訪問
    ```
5.  **啟動 Docker Compose 服務：**
    ```bash
    docker-compose up --build -d
    ```
    * `--build`：構建（或重新構建）映像檔。
    * `-d`：在後台運行容器。

## 網站訪問

成功啟動後，您應該可以透過以下 URL 訪問不同的 PHP 版本：

* **PHP 7.0 專案：** `http://php7-0.localhost/`
* **PHP 7.4 專案：** `http://php7-4.localhost/`
* **PHP 8.3 專案 (預設站點)：** `http://php8-3.localhost/` 或 `http://localhost/` (取決於 `nginx/conf.d/default.conf` 中的 `server_name` 配置)

## 資料庫與快取連線資訊 (在 PHP 應用程式內部)

您的 PHP 應用程式容器 (`php-fpm-7.0`, `php-fpm-7.4`, `php-fpm-8.3`) 應使用以下服務名稱和端口來連接資料庫和 Redis：

| 服務              | 主機 (Host)    | 端口 (Port) | 用戶名         | 密碼           | 資料庫名           |
| :---------------- | :------------- | :---------- | :------------- | :------------- | :----------------- |
| **MySQL** | `mysql`        | `3306`      | `${MYSQL_USER}` | `${MYSQL_PASSWORD}` | `${MYSQL_DATABASE}` |
| **PostgreSQL** | `postgres`     | `5432`      | `${POSTGRES_USER}`| `${POSTGRES_PASSWORD}`| `${POSTGRES_DB}`    |
| **Redis** | `redis`        | `6379`      | `N/A`          | `${REDIS_PASSWORD}` (如果設定) | `N/A`              |

**範例 (PHP 連線到 MySQL):**

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

## 常見問題與疑難排解

* **`502 Bad Gateway`**：
    * 檢查 `docker ps` 確認所有 PHP-FPM 和 Nginx 容器是否都在運行。
    * 檢查 `docker logs nginx` 和 `docker logs php-fpm-<version>` 以獲取錯誤訊息。
    * 確認 Nginx 配置（`nginx/conf.d/*.conf`）中的 `fastcgi_pass` 指令指向正確的 PHP-FPM 服務名稱和端口（應為 `php-fpm-X.X:9000`）。
    * 確認 PHP-FPM 的 `www.conf` 檔案中 `listen = 0.0.0.0:9000` 或 `listen = 9000`，確保它確實監聽在 9000 端口。 (目前提供的 `www.conf` 範例是 `127.0.0.1:9000`，這在 Docker 內部跨容器連線時可能會導致問題，建議修改為 `0.0.0.0:9000` 或直接 `9000`)。
* **PHP 擴展缺失：**
    * 如果您的應用程式需要特定擴展，請編輯對應 PHP 版本的 `Dockerfile` 並在 `RUN apt-get install -y` 和 `docker-php-ext-install` 行中添加它們。
    * 對於 PECL 擴展（例如 `imagick`, `redis`, `amqp`），請確保它們透過 `pecl install` 安裝後，並透過 `echo "extension=extension_name.so" > /usr/local/etc/php/conf.d/20-extension_name.ini` 方式啟用。
* **資料庫連線問題：**
    * 檢查 `docker logs mysql` / `docker logs postgres` / `docker logs redis` 查看資料庫/快取服務是否有啟動錯誤。
    * 確保您的 PHP 應用程式中使用正確的服務名稱 (`mysql`, `postgres`, `redis`) 和端口。
    * 檢查 `docker-compose.yml` 中資料庫服務的 `environment` 變數是否與應用程式中的憑證匹配。
* **資源不足：**
    * 如果您的電腦記憶體或 CPU 不足，多個服務運行可能會導致效能問題。考慮增加 Docker Desktop 的資源限制或關閉不必要的服務。

## 清理環境

要停止並移除所有服務、網路和數據卷（會清除所有資料庫數據），請運行：

```bash
docker-compose down --volumes
```

---
