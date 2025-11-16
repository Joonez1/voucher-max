# VoucherMax Mikrotik Router Integration Guide

## Overview

This guide provides comprehensive instructions for linking your VoucherMax application to Mikrotik routers, including terminal scripts, configuration commands, and best practices for deployment.

## Prerequisites

1. **Mikrotik Router**: RouterOS 6.40+ or RouterOS 7.x
2. **Network Access**: The VoucherMax server must be able to reach the Mikrotik router
3. **Admin Access**: Full administrative access to the Mikrotik router
4. **API Access**: Mikrotik API service enabled

## Part 1: Mikrotik Router Configuration

### 1.1 Enable API Service

Connect to your Mikrotik router and enable the API service:

```bash
# Via Winbox/WebFig GUI:
# IP > Services > API > Enable

# Via Terminal/CLI:
/ip service enable api
/ip service set api port=8728
```

### 1.2 Create API User

Create a dedicated user for VoucherMax API access:

```bash
# Create user group with necessary permissions
/user group add name=vouchermax-api policy=api,read,write,policy,test,winbox,local,telnet,ssh,ftp,reboot,sensitive

# Create dedicated API user
/user add name=vouchermax password=YOUR_SECURE_PASSWORD group=vouchermax-api

# Verify user creation
/user print
```

### 1.3 Configure Hotspot (if not already configured)

```bash
# Create hotspot profile
/ip hotspot profile add name="vouchermax-profile" hotspot-address=192.168.1.1 dns-name="vouchermax.local"

# Create IP pool for hotspot users
/ip pool add name=hotspot-pool ranges=192.168.1.100-192.168.1.200

# Setup hotspot on interface (adjust interface name as needed)
/ip hotspot add name=hotspot1 interface=wlan1 address-pool=hotspot-pool profile=vouchermax-profile

# Create user profile for vouchers
/ip hotspot user profile add name=voucher-profile rate-limit=2M/5M shared-users=1 keepalive-timeout=5m
```

### 1.4 Terminal Script for Bulk Voucher Creation

Save this script to create vouchers directly on the router:

```bash
#!/bin/bash
# bulk_create_vouchers.sh

# Configuration
ROUTER_IP="192.168.1.1"
USERNAME="vouchermax"
PASSWORD="YOUR_SECURE_PASSWORD"
PROFILE="voucher-profile"

# Generate multiple vouchers
for i in {1..100}
do
    VOUCHER_CODE="VOUCH$(printf "%04d" $i)"
    
    # Add user to hotspot
    /tool ssh-exec address=$ROUTER_IP user=$USERNAME command="/ip hotspot user add name=$VOUCHER_CODE password=$VOUCHER_CODE profile=$PROFILE"
    
    echo "Created voucher: $VOUCHER_CODE"
done

echo "Bulk voucher creation completed!"
```

## Part 2: VoucherMax Application Configuration

### 2.1 Environment Variables

Add these environment variables to your VoucherMax deployment:

```bash
# Mikrotik Router Configuration
MIKROTIK_DEFAULT_HOST=192.168.1.1
MIKROTIK_DEFAULT_PORT=8728
MIKROTIK_DEFAULT_USERNAME=vouchermax
MIKROTIK_DEFAULT_PASSWORD=YOUR_SECURE_PASSWORD

# Network Configuration
MIKROTIK_TIMEOUT=5000
MIKROTIK_KEEPALIVE=true
```

### 2.2 Admin Router Configuration

Each admin can configure their site-specific router through the admin dashboard:

1. **Login as Admin**: Use your admin credentials
2. **Navigate to Router Management**: Find "Mikrotik Router Management" section
3. **Add Router Configuration**:
   - Router Name: Site/Branch identifier
   - IP Address: Router's local IP
   - Username: API user created in step 1.2
   - Password: Secure password for API user
   - Port: 8728 (default API port)

### 2.3 Testing Connection

Use the built-in connection test feature:

```javascript
// Example test from VoucherMax admin dashboard
{
  "routerIp": "192.168.1.1",
  "username": "vouchermax",
  "password": "YOUR_SECURE_PASSWORD",
  "port": 8728
}
```

## Part 3: Network Security Configuration

### 3.1 Firewall Rules

Secure your Mikrotik router with proper firewall rules:

```bash
# Allow VoucherMax server access to API
/ip firewall filter add chain=input action=accept src-address=YOUR_VOUCHERMAX_SERVER_IP dst-port=8728 protocol=tcp comment="VoucherMax API Access"

# Block external access to API
/ip firewall filter add chain=input action=drop dst-port=8728 protocol=tcp comment="Block external API access"

# Allow hotspot users internet access
/ip firewall filter add chain=forward action=accept connection-state=established,related
/ip firewall filter add chain=forward action=accept src-address=192.168.1.0/24
```

### 3.2 SSL/TLS Configuration (Recommended)

For production environments, enable SSL for API access:

```bash
# Generate certificate
/certificate add name=api-cert common-name=vouchermax.local key-size=2048

# Sign certificate
/certificate sign api-cert

# Enable SSL API service
/ip service set api-ssl certificate=api-cert port=8729
/ip service enable api-ssl
```

## Part 4: Deployment Scripts

### 4.1 Automated Setup Script

Complete router setup script:

```bash
#!/bin/bash
# mikrotik_setup.sh

ROUTER_IP=$1
ADMIN_USER=$2
ADMIN_PASS=$3

if [ $# -ne 3 ]; then
    echo "Usage: $0 <router_ip> <admin_user> <admin_password>"
    exit 1
fi

echo "Setting up Mikrotik router at $ROUTER_IP..."

# Connect and configure router
ssh $ADMIN_USER@$ROUTER_IP << 'EOF'
# Enable API
/ip service enable api
/ip service set api port=8728

# Create VoucherMax user
/user group add name=vouchermax-api policy=api,read,write,policy,test,winbox,local,telnet,ssh,ftp,reboot,sensitive
/user add name=vouchermax password=SecureP@ssw0rd123 group=vouchermax-api

# Setup hotspot if needed
/ip hotspot user profile add name=voucher-profile rate-limit=2M/5M shared-users=1 keepalive-timeout=5m

# Configure firewall
/ip firewall filter add chain=input action=accept src-address=YOUR_VOUCHERMAX_SERVER_IP dst-port=8728 protocol=tcp comment="VoucherMax API Access"
/ip firewall filter add chain=input action=drop dst-port=8728 protocol=tcp comment="Block external API access"

echo "Mikrotik setup completed!"
EOF
```

### 4.2 Voucher Management Script

Script for ongoing voucher management:

```bash
#!/bin/bash
# manage_vouchers.sh

ROUTER_IP=$1
ACTION=$2
VOUCHER_CODE=$3

case $ACTION in
    "create")
        ssh vouchermax@$ROUTER_IP "/ip hotspot user add name=$VOUCHER_CODE password=$VOUCHER_CODE profile=voucher-profile"
        echo "Voucher $VOUCHER_CODE created"
        ;;
    "activate")
        ssh vouchermax@$ROUTER_IP "/ip hotspot user enable $VOUCHER_CODE"
        echo "Voucher $VOUCHER_CODE activated"
        ;;
    "deactivate")
        ssh vouchermax@$ROUTER_IP "/ip hotspot user disable $VOUCHER_CODE"
        echo "Voucher $VOUCHER_CODE deactivated"
        ;;
    "delete")
        ssh vouchermax@$ROUTER_IP "/ip hotspot user remove $VOUCHER_CODE"
        echo "Voucher $VOUCHER_CODE deleted"
        ;;
    *)
        echo "Usage: $0 <router_ip> <create|activate|deactivate|delete> <voucher_code>"
        ;;
esac
```

## Part 5: Monitoring and Maintenance

### 5.1 Health Check Script

Monitor router connectivity:

```bash
#!/bin/bash
# health_check.sh

ROUTERS=(
    "192.168.1.1"
    "192.168.2.1"
    "192.168.3.1"
)

for router in "${ROUTERS[@]}"
do
    if ping -c 1 $router &> /dev/null; then
        echo "✓ Router $router is online"
        
        # Check API service
        if nc -z $router 8728; then
            echo "✓ API service on $router is accessible"
        else
            echo "✗ API service on $router is not accessible"
        fi
    else
        echo "✗ Router $router is offline"
    fi
done
```

### 5.2 Log Monitoring

Monitor Mikrotik logs for voucher activities:

```bash
# View recent hotspot logs
/log print where topics~"hotspot"

# Monitor active sessions
/ip hotspot active print

# View user profiles
/ip hotspot user print
```

## Best Practices

### Security
1. **Change Default Passwords**: Always use strong, unique passwords
2. **Limit API Access**: Restrict API access to VoucherMax server IPs only
3. **Regular Updates**: Keep RouterOS updated to latest stable version
4. **Monitor Logs**: Regularly check system and access logs

### Performance
1. **Connection Pooling**: Use connection pooling for API calls
2. **Rate Limiting**: Implement rate limiting for voucher creation
3. **Caching**: Cache router status and configuration data
4. **Timeout Handling**: Set appropriate timeouts for API calls

### Scalability
1. **Load Balancing**: Use multiple routers for high-traffic locations
2. **Database Optimization**: Optimize voucher database queries
3. **Async Processing**: Use async processing for bulk operations
4. **Monitoring**: Implement comprehensive monitoring and alerting

## Troubleshooting

### Common Issues

1. **API Connection Failed**
   - Check router IP and credentials
   - Verify API service is enabled
   - Check firewall rules

2. **Voucher Creation Failed**
   - Verify user permissions
   - Check hotspot configuration
   - Ensure unique voucher codes

3. **Slow Performance**
   - Check network latency
   - Optimize API calls
   - Review router resource usage

### Support Commands

```bash
# Check API service status
/ip service print where name=api

# View system resources
/system resource print

# Check user permissions
/user print detail

# Monitor API connections
/ip service-port print
```

## Conclusion

This guide provides a comprehensive setup for integrating VoucherMax with Mikrotik routers. Follow the security best practices and monitoring guidelines for a robust, production-ready deployment.

For additional support, refer to the Mikrotik documentation or contact the VoucherMax support team.