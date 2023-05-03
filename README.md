# MySQL Scripts - Full Backups

This script is designed to create full backups of any MySQL databases located in containers, using the 'mysqldump' command. These backups can be used to restore the databases in the event of data loss or corruption.

## Using:

`/bin/mysql_backup.sh [CONTAINER 1 NAME] [CONTAINER 2 NAME] [CONTAINER ... NAME]`

## E.g.:
`/bin/mysql_backup.sh mysql_master mysql_slave1  mysql_slave2`

## Code:
```bash
#!/bin/bash

# Nome do arquivo de backup
BACKUP_FILENAME="mysql_$(date +"%FT%T").sql"

# Database Credentials (Username Only)
BACKUP_USER="username"
# Diretório do arquivo de backup geral
BACKUP_DIR=/path/to/backup/folder

# Database name
CONTAINER_NAMES=("$@")

# Get the password from the environment variable
if [ -z "$BACKUP_PASSWORD" ]; then
  echo -e "\nError: BACKUP_PASSWORD environment variable is not set."
  echo -e "Try using: \"export BACKUP_PASSWORD=\"password\"\"\n"
  exit 1
fi

# Loop through container names and execute backup for each container
for CONTAINER_NAME in "${CONTAINER_NAMES[@]}"
do  
  # Directory for container-specific backup file
  CONTAINER_BACKUP_DIR="$BACKUP_DIR/$CONTAINER_NAME"
  # Create directory if it doesn't exist
  mkdir -p "$CONTAINER_BACKUP_DIR"
  # Backup Command
  docker exec -it -u root $CONTAINER_NAME mysqldump --all-databases -u $BACKUP_USER -p$BACKUP_PASSWORD > "$CONTAINER_BACKUP_DIR/$DB_NAME_$BACKUP_FILENAME"
  
  # Remove files older than a period (standard 30 days)
  if [ -n "$(ls -A $BACKUP_DIR/$CONTAINER_NAME)" ]; then
    find $BACKUP_DIR/$CONTAINER_NAME -name "*.sql" -type f -mtime +30 -delete 
  fi
done
```

## Password not set:
```
Error: BACKUP_PASSWORD environment variable is not set.
Try using: "export BACKUP_PASSWORD="password""
```
