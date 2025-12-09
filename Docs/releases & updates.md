# Release Pipeline to dev,sta,app

## data pipeline release

lots changed, here are some notes on this release

**what changed**
- EC2 machines upgraded node18->20
- DMZ machine upgraded docker-compose from v1.2.3 to V2x
- added new S3 bucket and permissions (-ingest bucket, api.service needs r/w to -ingest and -images)
- added aliases to dmz and ec2 machines for shortcuts
- had trouble getting the right .env file on api servers
- had to copy .pem files to dmz servers (I think)
- lots of trouble deploying the api build
  - had trouble getting the .env file to the api server /srv/api/.env
  - the api machine ip address changed, so EC2_PRIVATE_IP secret on github had to update
- db update v0.0.17->v0.0.18:
  - added schemas: pipeline, stage, postgis
  - installed postgis and pg_tgrm (similarity) extensions

**convenience things, and stuff learned:**
- ec2 aliases: (la, jlog, jlogall, cdapi, cdapp, cdb, restart, getaliases, getupdatesh)
- dmz aliases: (app<env>, api<env>)
- local aliases: (awsdns $instanceId)
- can run docker-compose from the dmz machines (~/code/db-schema)

# Server maintenance/checking
```
ssh dmzdev|dmzsta|dmzapp
apipro # ssh -i app-key-pair.pem ec2-user@publicdnsname
node --version
curl http://localhost:3001/v1/version

# check on node version
ps aux | grep node
sudo ss -tulpn | grep node

## see what the last service errors were
elog # sudo tail /srv/api/logs/error.log

## monitor journal
jlog or jlogall # sudo journalctl -u api.service -f

restart # sudo systemctl restart api.service
```

## debugging
```
# 1. Check disk space
df -h

# 2. Check what's taking up space in /srv/app
sudo du -sh /srv/app/* | sort -h

# 3. Check for old app.zip files in /home/www-data
sudo du -sh /home/www-data/*

# 4. Clean up old app.zip (if it exists)
sudo rm -f /home/www-data/app.zip

# 5. Clean up old node_modules if there are multiple versions
# (This might help if there are leftover files from failed deployments)

# 6. Check for large log files
sudo du -sh /var/log/* | sort -h
sudo journalctl --disk-usage

# 7. Clean up old journal logs (keeps last 7 days)
sudo journalctl --vacuum-time=7d

# 8. Check for old npm cache
sudo du -sh ~/.npm 2>/dev/null || echo "No npm cache found"

```

## cp files from S3 to dev servers
```
cdms;./deployBootstrapScripts.sh

# SSH to your dev API server
cd /bootstrap
sudo aws s3 cp s3://dev-trippl-api-config/update.sh update.sh
sudo chmod +x update.sh
```