#!/bin/bash

# Specify the backup directory
backup_dir="/backup/"

# Function to perform the database backup for a user and database
perform_backup() {
    local user="$1"
    local db="$2"
    # Get the current date and time
    current_date=$(date +"%Y-%m-%d")
    current_time=$(date +"%H-%M-%S")

    # Backup filename with timestamp
    backup_filename="${backup_dir}backup_${current_date}_${current_time}_${user}_${db}.tar.gz"

    # Backup the database using cPanel's backup tool
    /usr/local/cpanel/bin/backup --user="$user" --output-dir="$backup_dir" --type=mysql --database="$db" --compression=gzip --file-prefix="$backup_filename"
}

# Store the count of existing backup files
existing_backups_count=$(find "$backup_dir" -type f -name "backup_*" | wc -l)

# Loop through each cPanel user
for user in $(ls -A /var/cpanel/users/); do
    # Loop through each MySQL database for the user
    for db in $(mysql -e "SHOW DATABASES;" | grep -Ev "^(Database|information_schema|performance_schema|sys)$"); do
        perform_backup "$user" "$db"
    done
done

# Cleanup old backups (older than 3 days)
deleted_backups_count=$(find "$backup_dir" -type f -name "backup_*" -mtime +3 -exec rm {} \; -print | wc -l)

# Send email notification if any backups were deleted during cleanup
if [ "$deleted_backups_count" -gt 0 ]; then
    echo "Important: $deleted_backups_count old backups were deleted during cleanup." | mailx -s "Local_Dev_Backup_Deletion_Warning (0.0.0.0)" xyz@example abc@example.com
fi

# Send email notification
echo "Daily_Backup_0.0.0.0 (Last 3days data backup availability)" | mailx -s "Local_Dev_Backup_Is_Successfully_Done (0.0.0.0)" xyz@example.com abc@example.com
