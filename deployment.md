# OSQuery + FleetDM Deployment Guide for macOS

## 1. FleetDM Server Setup

### Prerequisites
- A Linux server (Ubuntu 20.04+ recommended)
- Docker and Docker Compose
- Valid domain name with SSL certificate
- Minimum 4GB RAM, 2 CPU cores

### Server Installation

```bash
# Install Docker and Docker Compose if not already installed
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Create Fleet directory
mkdir ~/fleet && cd ~/fleet

# Create Docker Compose configuration
cat << 'EOF' > docker-compose.yml
version: '3'
services:
  mysql:
    image: mysql:8.0
    command: --default-authentication-plugin=mysql_native_password
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: your_secure_password
      MYSQL_DATABASE: fleet
      MYSQL_USER: fleet
      MYSQL_PASSWORD: fleet_secure_password
    volumes:
      - mysql-data:/var/lib/mysql

  fleet:
    image: fleetdm/fleet:latest
    ports:
      - "8080:8080"
    environment:
      FLEET_MYSQL_ADDRESS: mysql:3306
      FLEET_MYSQL_DATABASE: fleet
      FLEET_MYSQL_USERNAME: fleet
      FLEET_MYSQL_PASSWORD: fleet_secure_password
      FLEET_SERVER_ADDRESS: https://your.domain.com:8080
      FLEET_AUTH_JWT_KEY: random_secure_string_here
    depends_on:
      - mysql
    restart: always

volumes:
  mysql-data:
EOF

# Start Fleet
docker-compose up -d
```

### Initial Fleet Configuration

1. Access the Fleet UI at `https://your.domain.com:8080`
2. Create the initial admin account
3. Generate an enrollment secret from Settings → Enroll Secret

## 2. OSQuery Installation on macOS Clients

### Install OSQuery

```bash
# Install using Homebrew
brew install osquery

# Or download the package from osquery.io
# https://osquery.io/downloads/official/
```

### Install Fleet Launcher

```bash
# Download Fleet Launcher
curl -O https://github.com/fleetdm/fleet/releases/latest/download/fleet-launcher-darwin.pkg

# Install Fleet Launcher
sudo installer -pkg fleet-launcher-darwin.pkg -target /
```

### Configure Fleet Launcher

```bash
# Create configuration directory
sudo mkdir -p /etc/fleet

# Create launcher configuration
sudo cat << EOF > /etc/fleet/config
{
  "server": "https://your.domain.com:8080",
  "enroll_secret": "your_enrollment_secret_here",
  "root_directory": "/var/fleet",
  "logging_directory": "/var/log/fleet"
}
EOF

# Start Fleet Launcher service
sudo launchctl load /Library/LaunchDaemons/com.fleetdm.fleet-launcher.plist
```

## 3. Vulnerability Scanning Configuration

### Create Basic Queries in Fleet

Navigate to Queries → New Query and add these essential security queries:

```sql
-- Check for SIP Status
SELECT * FROM sip_config;

-- List all installed applications
SELECT name, path, bundle_version
FROM apps;

-- Check for FileVault status
SELECT * FROM disk_encryption;

-- Monitor SSH config
SELECT * FROM ssh_configs;

-- Check running processes
SELECT name, path, pid 
FROM processes 
WHERE on_disk = 0;
```

### Schedule Automated Scans

1. Create a new query pack in Fleet:
   - Navigate to Packs → New Pack
   - Name it "MacOS Security Scanning"
   - Add your queries with appropriate intervals

2. Configure automated query intervals:
   - Critical security checks: 1 hour
   - System inventory: 24 hours
   - Configuration checks: 6 hours

### Set Up Alerts

1. Configure alert rules in Fleet:
   - Navigate to Alerts → New Alert
   - Set conditions for critical findings
   - Configure notification channels (email, Slack, etc.)

2. Example alert conditions:
   - SIP disabled
   - FileVault disabled
   - Unauthorized SSH configurations
   - Unknown processes running

## 4. Best Practices

1. **Performance Optimization**
   - Stagger query execution times
   - Set appropriate intervals based on criticality
   - Monitor system resource usage

2. **Security Considerations**
   - Regularly rotate enrollment secrets
   - Use TLS for all communications
   - Implement least privilege access
   - Regular backup of Fleet configuration

3. **Maintenance**
   - Regular updates for OSQuery and Fleet
   - Monitor logs for errors
   - Regular review of query performance
   - Backup of configuration and data

## 5. Troubleshooting

### Common Issues and Solutions

1. Connection Issues:
```bash
# Check Fleet Launcher status
sudo launchctl list | grep fleet

# View logs
sudo tail -f /var/log/fleet/launcher.log
```

2. Query Failures:
```bash
# Check OSQuery status
sudo osqueryi --verbose
```

3. Performance Issues:
```bash
# Monitor resource usage
sudo osqueryi "SELECT * FROM processes WHERE name LIKE '%osquery%';"
```
