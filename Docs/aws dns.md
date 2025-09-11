unless you take specific steps, the private IP of an EC2 instance in a VPC can change on stop/start. Here’s the breakdown:

⸻

1️⃣ Private IP behavior
	•	Default behavior:
	•	EC2 instances in a default or non-custom VPC subnet usually get a dynamically assigned private IP.
	•	If you stop and start the instance, the private IP can change.
	•	Elastic/private IP reservation:
	•	If you assign a static private IP when launching the EC2, it will stay the same even after stop/start.
	•	You can do this in the VPC → Subnets → EC2 → Networking → Private IPs.

⸻

2️⃣ Public IP behavior
	•	If an instance has a public IP (not Elastic), that will always change on stop/start.
	•	An Elastic IP (public) stays the same — but in your case, EC2 stays private, so public IP isn’t needed.

⸻

3️⃣ Recommendations for your setup

Since you’re deploying via DMZ → EC2 private IP:
	1.	Assign a fixed private IP to each EC2 instance.
	•	When launching the EC2, select “Primary private IPv4 address”.
	•	Or modify an existing instance: stop it, assign the private IP in the VPC console, then start it.
	2.	Use environment secrets in GitHub Actions for that private IP.
	•	DEV_EC2_PRIVATE_IP, STAGING_EC2_PRIVATE_IP, PROD_EC2_PRIVATE_IP.
	•	This ensures deployments always hit the correct private IP, even if the instance is restarted.
	3.	Keep EC2 in private subnet
	•	Only the DMZ (jump host) has a public IP.
	•	Deployment still works via DMZ.

⸻

✅ Summary:
	•	Private IP can change on stop/start unless it’s statically assigned.
	•	For stable deployments, assign a static private IP for each EC2.
	•	No Elastic IP is needed for private EC2s — your DMZ handles public exposure.

⸻

If you want, I can also show a step-by-step on assigning a static private IP to an existing EC2 instance without breaking its networking. Do you want me to do that?

Perfect — here’s how to assign a static private IP to an existing EC2 instance in your VPC without breaking its networking:

⸻

Step 1: Stop the EC2 instance
	•	In the AWS Console → EC2 → Instances → Select your instance → Instance state → Stop
	•	The instance must be stopped to modify its primary private IP.

⸻

Step 2: Find the subnet and current private IP
	1.	Go to EC2 → Networking → Subnet.
	2.	Note the subnet ID and the current private IP (for reference).

⸻

Step 3: Assign a static private IP

Option A: Assign same IP as current (makes it stay consistent)
	1.	Go to EC2 → Networking → Manage IP Addresses (while instance is stopped).
	2.	Under Primary private IP, click Assign new IP or Set to specific IP.
	3.	Enter the IP you want (can be the current one or pick an unused IP in the subnet).
	4.	Save changes.

Option B: Pick a new IP (if the old one is already dynamic)
	•	Make sure it’s within the subnet CIDR range and not already used.

⸻

Step 4: Start the instance
	•	Start the EC2 → it now keeps the assigned private IP permanently.
	•	Update your GitHub Actions environment secret (DEV_EC2_PRIVATE_IP) to match this IP.

⸻

Step 5: Test SSH through DMZ
```bash
ssh -A DMZ_USER@DMZ_PUBLIC_IP
ssh EC2_USER@EC2_PRIVATE_IP
```

- It should connect successfully.
- Stop/start the EC2 again → private IP stays the same.

⸻

✅ Notes / best practices
	•	Keep EC2 in a private subnet (no public IP).
	•	DMZ handles public access and SSH.
	•	Target groups for ALB use the private IP → no changes needed.
	•	Document the assigned private IP per environment (dev/staging/prod) in GitHub Secrets.

⸻

If you want, I can update your workflow YAML so it uses these static private IPs automatically per environment, so deployments never break on stop/start. Do you want me to do that?