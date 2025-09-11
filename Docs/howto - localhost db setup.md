## Create and setup database (localhost)

```bash
# Create db - get postgres image and start container
cddb
docker-compose up -d trippl-db # defaults to --env-file .env and -f docker-compose.yml

# Run schema migrations
dbup   # which is an alias for docker-compose run --rm liquibase update


# Add Users
# this is the alias I created to call our api hook
# hook will add an account record and add 'user' role
# usage: newuser 'auth0Id' 'email' 'firstname' 'lastname'
alias newuser='f(){ curl -X POST -H "Content-Type: application/json" \
  -d "{ \"accessKey\": \"local-auth0-key\", \"auth0Id\": \"$1\", \"email\": \"$2\", \"firstName\": \"$3\", \"lastName\": \"$4\" }" \
  http://localhost:3001/v1/hooks/auth0; }; f'

# api-server has to be running
newuser "auth0|688654361932c27c22a7bfa2" "maureen@trippl.ca" "trippl" "super_admin";
newuser "auth0|6603c8e15a313d868b8e94de" "adamsbeach@gmail.com" "bookingpei" "admin";
newuser "auth0|6808aaf8b9d26a7cad58a920" "test@test.com" "test" "user";
newuser "auth0|68b37783bc92b37d9e83e85d" "pointseast@trippl.ca" "pointseast" "admin";

### Assign admin roles to users
# use sql query, easier than getting token for v1/assign-role endpont

ALTER TABLE rbac_account_roles
ADD CONSTRAINT unique_account_role UNIQUE (account_id, role_id);

INSERT INTO rbac_account_roles (account_id, role_id, granted_by)
SELECT a.id, r.id, '1'
FROM accounts a
JOIN rbac_roles r ON r.name = 'editor'
WHERE a.email = 'test@test.com'
ON CONFLICT (account_id, role_id) DO NOTHING;

INSERT INTO rbac_account_roles (account_id, role_id, granted_by)
SELECT a.id, r.id, '1' FROM accounts a JOIN rbac_roles r ON r.name = 'admin'
WHERE a.email IN ('adamsbeach@gmail.com', 'pointseast@trippl.ca')
  AND NOT EXISTS ( SELECT 1 FROM rbac_account_roles rr WHERE rr.account_id = a.id AND rr.role_id = r.id  );
INSERT INTO rbac_account_roles (account_id, role_id, granted_by)
SELECT a.id, r.id, '1' FROM accounts a JOIN rbac_roles r ON r.name = 'super_admin'
WHERE a.email IN ('maureen@trippl.ca')
  AND NOT EXISTS ( SELECT 1 FROM rbac_account_roles rr WHERE rr.account_id = a.id AND rr.role_id = r.id  );

### Set dmo owner

UPDATE accounts SET dmo_id = 4 WHERE email = 'pointseast@trippl.ca';
UPDATE accounts SET dmo_id = 5 WHERE email = 'adamsbeach@gmail.com';

### IMPORT LISTINGS
npm run import:local ALL  # [dmo|google/ca/ns|ALL]

```

## Build and deploy

```bash
# local development
lint | test | build

# before a pr make sure to replicate the github build
prbuild # npm run test:quiet; npm run ci:lint; npm run build

# Deploy db updates
tunnel dev|staging|app  # cdms; ./connectDatabaseTunnel.sh $1



```
