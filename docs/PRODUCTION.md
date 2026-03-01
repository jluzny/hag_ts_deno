# HAG HVAC - Production Deployment Guide

This guide covers deploying the HAG HVAC automation system to production.

## Production Configuration

### Config Files
- **Development**: `config/hvac_config_dev.yaml`
- **Production**: `config/hvac_config_prod.yaml`
- **Test**: `config/hvac_config_test.yaml`

### Key Production Differences
- **Log Level**: `info` instead of `debug` (reduced verbosity)
- **Retries**: Increased from 5 to 10 for better reliability
- **Retry Delay**: Increased from 1s to 2s for production stability
- **HVAC Entities**: All entities enabled by default
- **Temperature Thresholds**: Optimized for production comfort

## Deno Tasks

### Development
```bash
deno task dev              # Run with dev config and debug logging
deno task dev:watch        # Run with file watching
```

### Production
```bash
deno task prod             # Run with production config
deno task prod:build       # Build production binary
deno task prod:start       # Start production (optimized)
deno task prod:systemd     # Run with systemd integration
```

### Testing
```bash
deno task test             # Run all tests
deno task test:unit        # Run unit tests only
deno task test:integration # Run integration tests
deno task ci               # Run full CI pipeline
```

## Production Deployment

### Prerequisites
- Deno 1.40+ installed
- systemd-based Linux system
- Home Assistant accessible
- Valid Home Assistant token

### Automated Deployment

1. **Run the deployment script** (as root):
   ```bash
   sudo ./scripts/deploy-prod.sh
   ```

2. **Configure your Home Assistant token**:
   ```bash
   sudo systemctl edit hag-hvac
   ```
   Add:
   ```ini
   [Service]
   Environment=HASS_HassOptions__Token=your_actual_token_here
   ```

3. **Restart the service**:
   ```bash
   sudo systemctl restart hag-hvac
   ```

### Manual Deployment

1. **Create user and directories**:
   ```bash
   sudo useradd -r -s /bin/bash -d /opt/hag-hvac hag
   sudo mkdir -p /opt/hag-hvac
   ```

2. **Copy application files**:
   ```bash
   sudo cp -r src/ config/ deno.json deno.lock /opt/hag-hvac/
   sudo chown -R hag:hag /opt/hag-hvac
   ```

3. **Install systemd service**:
   ```bash
   sudo cp config/hag-hvac.service /etc/systemd/system/
   sudo systemctl daemon-reload
   sudo systemctl enable hag-hvac
   ```

4. **Configure environment variables**:
   ```bash
   sudo systemctl edit hag-hvac
   ```

5. **Start the service**:
   ```bash
   sudo systemctl start hag-hvac
   ```

## Production Monitoring

### Service Management
```bash
# Check service status
sudo systemctl status hag-hvac

# View logs
sudo journalctl -u hag-hvac -f

# Restart service
sudo systemctl restart hag-hvac

# Stop service
sudo systemctl stop hag-hvac
```

### Log Analysis
```bash
# Recent logs
sudo journalctl -u hag-hvac -n 50

# Logs since yesterday
sudo journalctl -u hag-hvac --since yesterday

# Error logs only
sudo journalctl -u hag-hvac -p err
```

## Configuration

### Environment Variables
- `HASS_HassOptions__Token`: Your Home Assistant long-lived access token

### Production Settings
The production config includes:
- **Optimized logging**: Only important events
- **Reliability**: Increased retries and delays
- **Security**: No AI enabled by default
- **Performance**: All HVAC entities enabled
- **Comfort**: Balanced temperature thresholds

### Temperature Thresholds
- **Heating**: 19.5°C - 20.5°C (indoor)
- **Cooling**: 23.5°C - 24.5°C (indoor)
- **Active Hours**: 6:00 AM - 11:00 PM

## Security Considerations

1. **Token Security**: Store Home Assistant tokens in environment variables
2. **File Permissions**: Ensure proper file ownership (`hag:hag`)
3. **Network Security**: Use HTTPS for Home Assistant if possible
4. **Log Security**: Logs may contain sensitive information

## Troubleshooting

### Common Issues

1. **Service won't start**:
   - Check token configuration
   - Verify Home Assistant connectivity
   - Check file permissions

2. **No HVAC control**:
   - Verify entity IDs in config
   - Check Home Assistant entity states
   - Ensure entities are enabled

3. **Temperature not updating**:
   - Check sensor entity IDs
   - Verify sensor availability in Home Assistant
   - Check network connectivity

### Debug Mode

For debugging in production:
```bash
# Temporary debug mode
sudo systemctl edit hag-hvac
```
Add:
```ini
[Service]
Environment=LOG_LEVEL=debug
```

Then restart:
```bash
sudo systemctl restart hag-hvac
```

## Performance

### Resource Usage
- **Memory**: ~50MB typical usage
- **CPU**: Very low, event-driven
- **Network**: Minimal (WebSocket + occasional REST calls)

### Optimization Features
- **Efficient logging**: Debug info only when needed
- **Smart evaluation**: Only evaluate when temperatures change
- **Connection pooling**: Reuse Home Assistant connections
- **Graceful degradation**: Continue operation during temporary issues

## Backup & Recovery

### Configuration Backup
```bash
# Backup configuration
sudo cp -r /opt/hag-hvac/config /backup/hag-hvac-config-$(date +%Y%m%d)

# Restore configuration
sudo cp -r /backup/hag-hvac-config-20240101/* /opt/hag-hvac/config/
sudo systemctl restart hag-hvac
```

### Database Backup
The system is stateless - no database backup needed. All state is managed by Home Assistant.

## Updates

### Application Updates
```bash
# Stop service
sudo systemctl stop hag-hvac

# Update application files
sudo cp -r /path/to/new/version/* /opt/hag-hvac/

# Fix permissions
sudo chown -R hag:hag /opt/hag-hvac

# Start service
sudo systemctl start hag-hvac
```

### Deno Updates
```bash
# Update Deno
curl -fsSL https://deno.land/install.sh | sh

# Update dependencies
cd /opt/hag-hvac
sudo -u hag deno cache --reload src/main.ts
```