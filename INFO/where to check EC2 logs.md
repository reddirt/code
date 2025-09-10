Where to Check Logs on Your EC2 Instances:
Based on your infrastructure setup, the API server runs as a systemd service. Here's how to check the logs:
1. Check API Service Logs:
```bash
# SSH into your dev EC2 instance, then:
sudo journalctl -u api.service -f

# Or check recent logs:
sudo journalctl -u api.service --since "1 hour ago"

# Or check logs from today:
sudo journalctl -u api.service --since today
```

2. Check Nginx Logs:
```bash
# Access logs (shows incoming requests):
sudo tail -f /var/log/nginx/access.log

# Error logs (shows nginx errors):
sudo tail -f /var/log/nginx/error.log
```

3. Check System Logs:
```bash
# General system logs:
sudo journalctl -f

# Check for any application crashes:
sudo journalctl -p err
```