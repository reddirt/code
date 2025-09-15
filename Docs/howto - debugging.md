# Debugging
## Check Recent Log Entries
```bash
# View the last 50 lines of the error log
tail -50 /srv/api/logs/error.log

# View real-time log entries
tail -f /srv/api/logs/error.log
```

## Look for 500 Errors
```bash
# Search for 500 errors in the log
grep -i "500\|error\|exception" /srv/api/logs/error.log | tail -20
```

## Check for Coordinate Validation Errors
```bash
# Look for latitude/longitude validation errors
grep -i "latitude\|longitude\|coordinate" /srv/api/logs/error.log | tail -10
```

## service status
```bash
sudo systemctl status api.service

# restart
cd /srv/api
sudo systemctl restart api.service
```