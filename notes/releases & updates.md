# database

## localhost
- run docker database

PR to main

tunnel dev|staging|app

docker-compose run --rm liquibase status

whatif: liquibase.exception.ValidationFailedException: Validation Failed:

docker-compose run --rm liquibase clearCheckSums

docker-compose run --rm liquibase update

# dev.trippl.ca
```
ssh dmzdev

**api-server**
# files are in /srv/api

ssh -i dev ...pem ec2-user@publicdnsname

# check on node and the service
ps aux | grep node
sudo ss -tulpn | grep node
curl http://localhost:3001/v1/version

# see what the last service errors were
sudo tail /srv/api/logs/error.log

# monitor stdout
sudo journalctl -u api.service -f

sudo systemctl restart api.service
```

# app-web
```
ssh -i dev ...pem ec2-user@publicdnsname


sudo mkdir -p /srv/app/logs
sudo chown -R www-data:www-data /srv/app/logs
```