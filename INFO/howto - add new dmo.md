# New DMO
### 🕋 Database updates
```bash
# manually add new dmo info to database tables:
- dmos (includes dmo hostname)
- dmo_info

# Create or choose a user account to be used as dmo owner account

# Set account as dmo owner
UPDATE accounts SET dmo_id = 'nn' WHERE email = 'dmoowner@email.com';

# Give account admin role
INSERT INTO rbac_account_roles (account_id, role_id, granted_by)
SELECT a.id, r.id, '1' FROM accounts a JOIN rbac_roles r ON r.name = 'admin'
WHERE a.email IN ('dmoowner@email.com')
  AND NOT EXISTS ( SELECT 1 FROM rbac_account_roles rr WHERE rr.account_id = a.id AND rr.role_id = r.id  );
```
### API Release
- release a new api-server
  - add hostname to corsOrigin in api-server/src/config/app.ts

  **TODO:** check to see if Auth0 can handle this with allowed cors Urls

### 🔐 SSL cert
- goto AWS Certificate Manager: issue a certificate
- create records in Route53 (DNS validation)
- or send CNAME key and value to dmo client to add to their zone records
- goto AWS EC2 - Load Balancers: edit app and api, edit HTTPS:443, certificates tab, Add Certificate, add pending.

### 🔑 Auth0 login/logout privledge
- goto manage.auth0.com, trippl-app PRODUCTION tenant
- goto Applications : Applications : trippl SPA
- update Application URIs section with new url


### ➡️ Import listings
```bash
# create .csv -> /scripts/_sourceData/dmo_name/data.csv

cdapi # is this required?
npm run transform:csv 18 ca-ns-valleyren

# local import
npm run import:local ca-ns-valleyren

# import to the env you want listings on: dev|staging|app
tunnel dmzapp  # hostname is in ~/.ssh/config
npm run import:app ca-ns-valleyren
```

### 📍 Add location id
```bash
# once you have locations, pick one and make it the default for the initial map load
# TODO this will change in the future

UPDATE dmos
SET location_id = (
  SELECT location_id
  FROM listings
  WHERE name = 'Avondale Golf Course'
)
WHERE name = 'ca-pe-pointseast';
```

### 