Here's an outline of shell and Python scripts to address each of these requirements:

### 1. Find and delete files older than 30 days
```bash
#!/bin/bash
# Find and delete files older than 30 days in the specified directory
DIRECTORY="/path/to/directory"
find "$DIRECTORY" -type f -mtime +30 -exec rm -f {} \;
```

### 2. Monitor disk usage, log details, and send an alert if usage exceeds 80%
```bash
#!/bin/bash
# Check disk usage and log/send an alert if usage exceeds 80%
THRESHOLD=80
LOGFILE="/path/to/disk_usage.log"
EMAIL="your-email@example.com"

usage=$(df / | grep / | awk '{ print $5 }' | sed 's/%//g')

if [ "$usage" -gt "$THRESHOLD" ]; then
  echo "Disk usage is at $usage%" >> "$LOGFILE"
  echo "Disk usage on the server has exceeded $THRESHOLD%. Current usage: $usage%" | mail -s "Disk Usage Alert" "$EMAIL"
fi
```

### 3. Rename `.txt` files by appending the current date
```bash
#!/bin/bash
# Rename .txt files to include the current date
DIRECTORY="/path/to/directory"
DATE=$(date +%Y-%m-%d)

for file in "$DIRECTORY"/*.txt; do
  [ -e "$file" ] || continue
  mv "$file" "${file%.txt}_$DATE.txt"
done
```

### 4. Check if a service is running, restart if not, and log the action
```bash
#!/bin/bash
# Check if httpd service is running and restart if not
SERVICE="httpd"
LOGFILE="/path/to/service_monitor.log"

if ! systemctl is-active --quiet "$SERVICE"; then
  echo "$(date): $SERVICE was down, restarting now" >> "$LOGFILE"
  systemctl restart "$SERVICE"
fi
```

### 5. Monitor CPU and memory usage, log if limits are exceeded
```bash
#!/bin/bash
# Log CPU and memory usage if limits are exceeded
LOGFILE="/path/to/resource_monitor.log"
CPU_THRESHOLD=90
MEM_THRESHOLD=75

while true; do
  CPU_USAGE=$(top -bn1 | grep "Cpu(s)" | sed "s/.*, *\([0-9.]*\)%* id.*/\1/" | awk '{print 100 - $1}')
  MEM_USAGE=$(free | grep Mem | awk '{print $3/$2 * 100.0}')

  if (( $(echo "$CPU_USAGE > $CPU_THRESHOLD" | bc -l) )) || (( $(echo "$MEM_USAGE > $MEM_THRESHOLD" | bc -l) )); then
    echo "$(date): CPU: $CPU_USAGE%, Memory: $MEM_USAGE%" >> "$LOGFILE"
  fi
  sleep 60
done
```

### 6. Perform and compress a directory backup with the current date, schedule with cron
```bash
#!/bin/bash
# Backup and compress a directory with the current date in the filename
SOURCE_DIR="/path/to/directory"
BACKUP_DIR="/path/to/backups"
DATE=$(date +%Y-%m-%d)
tar -czf "$BACKUP_DIR/backup_$DATE.tar.gz" "$SOURCE_DIR"
```
*Add this to cron to run daily at midnight:*
```bash
0 0 * * * /path/to/backup_script.sh
```

### 7. Check website availability, log and alert if unreachable
```bash
#!/bin/bash
# Check if websites are reachable, log if not
URLS=("https://example1.com" "https://example2.com")
LOGFILE="/path/to/website_monitor.log"
EMAIL="your-email@example.com"

for url in "${URLS[@]}"; do
  if ! curl -Is "$url" | head -n 1 | grep "200\|301\|302" > /dev/null; then
    echo "$(date): $url is unreachable" >> "$LOGFILE"
    echo "$url is unreachable" | mail -s "Website Unreachable Alert" "$EMAIL"
  fi
done
```

### 8. List all S3 buckets and their sizes using boto3
```python
import boto3

def get_s3_buckets_sizes():
    s3 = boto3.client('s3')
    buckets = s3.list_buckets()['Buckets']

    for bucket in buckets:
        bucket_name = bucket['Name']
        total_size = 0
        objects = s3.list_objects_v2(Bucket=bucket_name)

        if 'Contents' in objects:
            for obj in objects['Contents']:
                total_size += obj['Size']
        print(f"Bucket: {bucket_name}, Size: {total_size / (1024 * 1024):.2f} MB")

get_s3_buckets_sizes()
```

### 9. Start and stop EC2 instances based on a schedule
```bash
#!/bin/bash
# Start EC2 instances at 8 AM and stop at 8 PM using boto3
import boto3
from datetime import datetime

def manage_ec2_instances():
    ec2 = boto3.client('ec2')
    instances = ['i-1234567890abcdef0', 'i-0abcdef1234567890']  # Replace with your instance IDs
    current_hour = datetime.now().hour

    if 8 <= current_hour < 20:
        ec2.start_instances(InstanceIds=instances)
        print("Instances started")
    else:
        ec2.stop_instances(InstanceIds=instances)
        print("Instances stopped")

manage_ec2_instances()
```
*To schedule this script, add it to a cron job running every hour.*

### 10. Monitor directory and delete files larger than 100 MB
```bash
#!/bin/bash
# Monitor directory and delete files larger than 100 MB
DIRECTORY="/path/to/directory"
find "$DIRECTORY" -type f -size +100M -exec rm -f {} \;
```

These scripts cover the specified tasks, providing logging, alerts, and automation functionalities to support efficient server and file management.
