# 🔧 Snipe-IT Docker Installation & Backup Automation Pipeline

This project provides a fully automated setup for deploying [Snipe-IT](https://snipeitapp.com/) using Docker Compose, along with a scheduled backup routine for both the database and uploaded files.

---

## 📁 Project Structure

snipeit-pipeline/
├── .env
├── docker-compose.yml
├── docker/
│ └── backup.sh
├── scripts/
│ └── deploy.sh
└── backups/


---

## ✅ Features

- One-command installation
- `.env`-based configuration
- Scheduled automatic backups (MySQL + uploads)
- Ready for extension with cloud sync (AWS S3, Google Drive, etc)

---

## ⚙️ Prerequisites

- [Docker](https://www.docker.com/products/docker-desktop)
- [Docker Compose](https://docs.docker.com/compose/)
- Unix-based OS (Linux/macOS)

---

## 📦 1. Environment File

Create a `.env` file:

```env
MYSQL_ROOT_PASSWORD=rootpass
MYSQL_DATABASE=snipeit
MYSQL_USER=snipeuser
MYSQL_PASSWORD=snipepass
APP_KEY=base64:REPLACE_WITH_GENERATED_KEY
APP_URL=http://localhost:8080
TZ=America/Toronto


Generate a valid APP_KEY:

docker run --rm snipe/snipe-it php artisan key:generate --show

🐳 2. Docker Compose Configuration
Create docker-compose.yml:

version: '3.8'

services:
  mysql:
    image: mysql:5.7
    container_name: snipe-mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: ${MYSQL_DATABASE}
      MYSQL_USER: ${MYSQL_USER}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD}
    volumes:
      - mysql_data:/var/lib/mysql

  snipeit:
    image: snipe/snipe-it:latest
    container_name: snipe-app
    depends_on:
      - mysql
    restart: always
    ports:
      - "8080:80"
    environment:
      APP_ENV: production
      APP_DEBUG: false
      APP_KEY: ${APP_KEY}
      APP_URL: ${APP_URL}
      DB_HOST: mysql
      DB_DATABASE: ${MYSQL_DATABASE}
      DB_USERNAME: ${MYSQL_USER}
      DB_PASSWORD: ${MYSQL_PASSWORD}
      MAIL_PORT: 1025
      TZ: ${TZ}
    volumes:
      - uploads:/var/www/html/public/uploads

volumes:
  mysql_data:
  uploads:


🚀 3. Deployment Script
Create scripts/deploy.sh:

#!/bin/bash

set -e

echo "Starting Snipe-IT installation pipeline..."

source .env

docker compose up -d --build

echo "Snipe-IT is now running at $APP_URL"


Make it executable:
chmod +x scripts/deploy.sh

Run:
./scripts/deploy.sh

🔄 4. Backup Script
Create docker/backup.sh:
#!/bin/bash

TIMESTAMP=$(date +"%F_%H-%M")
BACKUP_DIR=./backups/${TIMESTAMP}
MYSQL_CONTAINER=snipe-mysql

echo "Starting backup to ${BACKUP_DIR}..."

mkdir -p "${BACKUP_DIR}"

# Database dump
docker exec ${MYSQL_CONTAINER} sh -c 'exec mysqldump -u root -p"$MYSQL_ROOT_PASSWORD" snipeit' > "${BACKUP_DIR}/snipeit.sql"

# Uploads backup
docker run --rm \
  -v snipeit-pipeline_uploads:/data:ro \
  -v "$(pwd)/backups":/backup \
  alpine \
  tar czf "/backup/${TIMESTAMP}/uploads.tar.gz" -C /data .

echo "Backup completed: ${BACKUP_DIR}"

Make it executable:
chmod +x docker/backup.sh

🕒 5. Optional: Daily Backup via Cron
Edit your crontab:
crontab -e

Add this line to run the backup every day at 2 AM:
0 2 * * * /full/path/to/docker/backup.sh >> /full/path/to/logs/backup.log 2>&1

☁️ 6. Optional Cloud Sync
Upload backups to AWS S3 or other cloud services.

AWS S3
aws s3 cp backups/ s3://your-snipeit-bucket/ --recursive

Using rclone (Google Drive, Dropbox, etc.)
rclone copy backups remote:your-backup-folder

🔐 Security Recommendations
Do not commit your .env file

Use GitHub Secrets or CI/CD secret management

In production: set up HTTPS, firewall, and limit container access

🧪 Local Testing
./scripts/deploy.sh     # Install & run
./docker/backup.sh      # Run manual backup

Access Snipe-IT at: http://localhost:8080

📬 Contribution & License
Feel free to fork or improve this project.
Licensed under MIT.

