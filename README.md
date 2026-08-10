# proxmox-obsidian-livesync

## Items required for procedure
1. Domain purchase (costs $15-$20)
2. Cloudflare account (free)
3.  

## Procedure
### Proxmox environment setup
1. Within proxmox, create an lxc container
<br>
In GUI, 'Create CT'
<br>
Set password
<br>
Under templates, select 'ubuntu-2404'
<br>
Disk size, '50GB'
<br>
CPU, 1 core [Default]
<br>
Memory, 512MB [Default]
<br>
Network, set:
<br>
Static IP: 10.0.0.252/24
<br>
Default Gateway: 10.0.0.1
<br>
Select option: "Start after created"
<br>
*NOTE* Ensure you have a containerised instance of Ubuntu 24.04 available, testing has not been performed on a 26.04 ubuntu instance

### Containerised couchdb instance
2. Update repositories and install updates
```bash
apt update && apt upgrade -y
```

### Docker installation
3. Install Docker
```bash
apt install -y ca-certificates curl gnupg
```
```bash
install -m 0755 -d /etc/apt/keyrings
```
```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```
```bash
chmod a+r /etc/apt/keyrings/docker.gpg
```
```bash
echo \ "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntui \$(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \tee /etc/apt/sources.list.d/docker.list > /dev/null
```
```bash
apt update
```
```bash
apt install -y docker-ce docker-ce-cli containerd.io dockerbuildx-compose-plugin
```
### Docker configuration
4. 
```bash
echo "services:
  obsidian-livesync:
     container_name: obsidian_livesync
    image: couchdb:latest
    env_file:
      - .env
    environment:
      - COUCHDB_USER=${COUCHDB_USER}
      - COUCHDB_PASSWORD=${COUCHDB_PASSWORD
    volumes:
      - /opt/couchdb/data:/opt/couchdb/data
      - /opt/couchdb/etc/local.d:/opt/couchdb/etc/local.d
    ports:
      - 5984:5984
    restart: unless-stopped
    healthcheck:
      test: curl --fail -s -u ${COUCHDB_USER}:${COUCHDB_PASSWORD} http://localhost:5984/_up | grep -Eo '"status:"ok"' || exit 1
      interval: 30s
      timeout: 10s
      retries: 3" >> compose.yaml
```
*NOTE* Momentarily you will use the compose file to prepare your dockerised couchdb instance
5. Prepare an .env file, it should look like the following:
```bash
vim .env
```
```
COUCHDB_USER: admin
COUCHDB_PASSWORD: <password>
```
press shift+:
<br>
type wq for write and quit
:wq

### Couchdb installation
6. Start your docker compose file
**Option 1:** Make the script included in this repo executable then run the script
```bash
chmod +x 
./start-container.sh
```
**OR**
**Option 2:** Input the line below
```bash
docker compose -f compose.yaml --env-file .env up -d
```

### Couchdb configuration
7. In a web browser, visit the following location: <ip for container>:5984/_utils
8. Visit "Setup"
9. Set "Configure a single node"
10. Input the same username and password as seen in the .env file
11. Go to "Verify", then "Verify Installation"
12. Create Database, then input name "Obsidian", stick with "non-partitioned"
**You can skip this step, obsidian will do it for you** 
13. Visit Configuration, then input the following:
Section      Name                 Value
chttpd       require_valid_user   true
chttpd_auth  require_valid_user   true
httpd        WWW-Authenticate     Basic realm="couchdb"
httpd        enable_cors          true
chttpd       enable_cors          true
chttpd       max_http_request_size 4294967296 
couchdb      max_document_size     50000000
cors         credentials           true
cors         origins               app://obsidian.md, capacitor://localhost, http://localhost

### Domain set-up

### Domain configuration

### Cloudflare tunnel setup
In a web browser, access your cloudflare dashboard and set up a cloudflare tunnel
<br>
Input your chosen subdomain and domain, your path if necessary.
<br>
*IMPORTANT* Your service url must be the following: http://couchdb:5984

### Community plugin Livesync setup

### Community plugin Livesync configuration

### Setting up community plugin livesync on mobile devices
