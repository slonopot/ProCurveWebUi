# HP ProCurve Switch Console API

A JavaScript API for managing HP ProCurve switches directly from the browser console. This API provides programmatic access to port management, VLAN configuration, and stack management functions.

## 🚀 Quick Start

1. **Open the HP ProCurve Switch Controller** in your browser
2. **Press F12** to open Developer Tools
3. **Go to Console tab**
4. **Connect to your switch**:
   ```javascript
   await HPAPI.connect('192.168.1.100')
   ```

## 📋 Table of Contents

- [Connection Management](#connection-management)
- [Port Management](#port-management)
- [VLAN Management](#vlan-management)
- [Stack Management](#stack-management)
- [Utilities](#utilities)
- [Examples](#examples)
- [Error Handling](#error-handling)

## 🔌 Connection Management

### Connect to Switch
```javascript
// Connect to switch
await HPAPI.connect('192.168.1.100')
// Returns: { success: true, message: "Connected to 192.168.1.100", portCount: 48 }
```

## 🔧 Port Management

### List All Ports
```javascript
// Get all ports with status
const ports = await HPAPI.ports.list()
console.log(ports)
/* Returns:
{
  "1": {
    "portType": "GigT",
    "enabled": true,
    "connected": true,
    "usage": 15,
    "speed": "1000Mbs"
  },
  "2": {
    "portType": "GigT", 
    "enabled": false,
    "connected": false,
    "usage": 0,
    "speed": "10Mbs"
  }
  // ... more ports
}
*/
```

### Enable/Disable Ports
```javascript
// Enable single port
await HPAPI.ports.enable(1)

// Enable multiple ports
await HPAPI.ports.enable([1, 2, 3, 4])

// Disable single port  
await HPAPI.ports.disable(5)

// Disable multiple ports
await HPAPI.ports.disable([10, 11, 12])
```

### Get Port Usage Statistics
```javascript
// Get real-time usage data
const usage = await HPAPI.ports.usage()
console.log(usage)
/* Returns:
{
  "1": {
    "connected": true,
    "usage": 25,
    "unicast": 20,
    "nonUnicast": 3,
    "errors": 2,
    "speed": "1000Mbs",
    "linkStatus": "G"
  }
  // ... more ports
}
*/
```

## 🏷️ VLAN Management

### List All VLANs
```javascript
const vlans = await HPAPI.vlans.list()
console.log(vlans)
/* Returns:
[
  {
    "id": "1",
    "name": "DEFAULT_VLAN (Primary)",
    "type": "STATIC",
    "taggedPorts": "",
    "untaggedPorts": "1-43, 45-48",
    "forbiddenPorts": ""
  },
  {
    "id": "100", 
    "name": "Management",
    "type": "STATIC",
    "taggedPorts": "1,2",
    "untaggedPorts": "",
    "forbiddenPorts": "3-48"
  }
]
*/
```

### Create VLANs
```javascript
// Create simple VLAN
await HPAPI.vlans.create(200, 'TestVLAN')

// Create VLAN with port configuration
await HPAPI.vlans.create(201, 'MyVLAN', {
  '1': 1,    // Port 1 = Tagged
  '2': 2,    // Port 2 = Untagged  
  '3': 3,    // Port 3 = Forbidden
  '4': 4     // Port 4 = Auto
})

// Port modes:
// 1 = Tagged
// 2 = Untagged
// 3 = Forbidden
// 4 = Auto (GVRP)
```

### Configure VLAN Ports
```javascript
// Configure ports for existing VLAN
await HPAPI.vlans.configurePorts(100, {
  '5': 2,    // Port 5 = Untagged
  '6': 1,    // Port 6 = Tagged
  '7': 3     // Port 7 = Forbidden
})

// Get current port configuration
const config = await HPAPI.vlans.getPorts(100)
console.log(config)
/* Returns:
{
  "5": { "mode": 2, "type": "untagged" },
  "6": { "mode": 1, "type": "tagged" },
  "7": { "mode": 3, "type": "forbidden" }
}
*/
```

### Primary VLAN Management
```javascript
// Set VLAN as Primary (handles untagged traffic)
await HPAPI.vlans.setPrimary(100)

// Note: Only one VLAN can be Primary at a time
```

### Delete VLANs
```javascript
// Delete regular VLAN
await HPAPI.vlans.delete(200)

// Delete Primary VLAN (requires force flag)
await HPAPI.vlans.delete(1, true)

// The API will automatically:
// 1. Remove all ports from the VLAN
// 2. Delete the VLAN
// 3. Warn if trying to delete Primary VLAN
```

## 🔗 Stack Management

### List Stack Members
```javascript
const members = await HPAPI.stack.members()
console.log(members)
/* Returns:
[
  {
    "switchNum": "1",
    "macAddr": "00:1B:78:12:34:56", 
    "systemName": "Switch-01",
    "deviceType": "ProCurve 2810-24G",
    "status": "Member Up"
  }
]
*/
```

### List Stack Candidates
```javascript
const candidates = await HPAPI.stack.candidates()
console.log(candidates)
/* Returns:
[
  {
    "macAddr": "00:1B:78:12:34:57",
    "systemName": "Switch-02", 
    "deviceType": "ProCurve 2810-24G"
  }
]
*/
```

### Add/Remove Stack Members
```javascript
// Add member to stack
await HPAPI.stack.addMember('00:1B:78:12:34:57', 'password123')

// Remove member from stack  
await HPAPI.stack.removeMember('2')
```

## 🛠️ Utilities

### Test Connection
```javascript
// Quick ping test
const isOnline = await HPAPI.utils.ping()
console.log(isOnline) // true/false
```

### Get Switch Information
```javascript
const info = await HPAPI.utils.info()
console.log(info)
/* Returns:
{
  "switchIP": "192.168.1.100",
  "portCount": 48,
  "vlanCount": 5,
  "stackMembers": 2,
  "connected": true
}
*/
```

## 💡 Examples

### Complete Port Management
```javascript
// Connect and manage ports
await HPAPI.connect('192.168.1.100')

// Get all ports
const ports = await HPAPI.ports.list()

// Find disabled ports
const disabledPorts = Object.entries(ports)
  .filter(([num, port]) => !port.enabled)
  .map(([num]) => num)

console.log('Disabled ports:', disabledPorts)

// Enable all disabled ports
if (disabledPorts.length > 0) {
  await HPAPI.ports.enable(disabledPorts)
  console.log(`Enabled ${disabledPorts.length} ports`)
}
```

### VLAN Batch Operations
```javascript
// Create multiple VLANs
const vlansToCreate = [
  { id: 10, name: 'Management', ports: {'1': 1, '2': 1} },
  { id: 20, name: 'Users', ports: {'3': 2, '4': 2, '5': 2} },
  { id: 30, name: 'Servers', ports: {'6': 1, '7': 1} }
]

for (const vlan of vlansToCreate) {
  await HPAPI.vlans.create(vlan.id, vlan.name, vlan.ports)
  console.log(`Created VLAN ${vlan.id}`)
}

// List all VLANs
const allVlans = await HPAPI.vlans.list()
console.table(allVlans)
```

### Port Usage Monitoring
```javascript
// Monitor high usage ports
async function monitorPorts() {
  const usage = await HPAPI.ports.usage()
  
  const highUsagePorts = Object.entries(usage)
    .filter(([num, data]) => data.usage > 80)
    .map(([num, data]) => ({ port: num, usage: data.usage }))
  
  if (highUsagePorts.length > 0) {
    console.warn('High usage ports:', highUsagePorts)
  }
  
  return highUsagePorts
}

// Run every 30 seconds
setInterval(monitorPorts, 30000)
```

### Bulk Port Configuration
```javascript
// Configure ports 1-10 for Management VLAN
const managementPorts = {}
for (let i = 1; i <= 10; i++) {
  managementPorts[i] = 2 // Untagged
}

await HPAPI.vlans.configurePorts(10, managementPorts)
console.log('Configured ports 1-10 for Management VLAN')
```

## ⚠️ Error Handling

All API functions throw errors that you should handle:

```javascript
try {
  await HPAPI.vlans.delete(1) // Try to delete Primary VLAN
} catch (error) {
  console.error('Error:', error.message)
  // Error: Cannot delete Primary VLAN. Use force=true to override or change Primary VLAN first.
}

// Proper error handling
async function safePortEnable(portNumbers) {
  try {
    const result = await HPAPI.ports.enable(portNumbers)
    console.log('✅ Success:', result)
    return result
  } catch (error) {
    console.error('❌ Failed to enable ports:', error.message)
    return { success: false, error: error.message }
  }
}

// Use with error handling
const result = await safePortEnable([1, 2, 3])
if (result.success) {
  console.log('Ports enabled successfully')
} else {
  console.log('Failed to enable ports')
}
```

## 🔒 CORS and Security

**Important**: Modern browsers block cross-origin requests (CORS). To use this API:

### Option 1: Chrome with Disabled Security (Development Only)
```bash
# Windows
chrome.exe --user-data-dir="C:/temp" --disable-web-security

# Mac
open -n -a /Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --args --user-data-dir="/tmp/chrome_dev_test" --disable-web-security

# Linux
google-chrome --user-data-dir="/tmp/chrome_dev_test" --disable-web-security
```

### Option 2: Local Proxy Server
Set up a local proxy to forward requests to your switch.

### Option 3: Browser Extension
Use a CORS-disabling browser extension (development only).

## 📝 Advanced Usage

### Chaining Operations
```javascript
// Complex workflow
async function setupNetworkSegmentation() {
  // Connect to switch
  await HPAPI.connect('192.168.1.100')
  
  // Create VLANs for different departments
  await HPAPI.vlans.create(10, 'Management')
  await HPAPI.vlans.create(20, 'Sales') 
  await HPAPI.vlans.create(30, 'Engineering')
  await HPAPI.vlans.create(40, 'Guest')
  
  // Configure Management VLAN (ports 1-4 tagged)
  const mgmtPorts = { '1': 1, '2': 1, '3': 1, '4': 1 }
  await HPAPI.vlans.configurePorts(10, mgmtPorts)
  
  // Configure Sales VLAN (ports 5-14 untagged)
  const salesPorts = {}
  for (let i = 5; i <= 14; i++) {
    salesPorts[i] = 2 // Untagged
  }
  await HPAPI.vlans.configurePorts(20, salesPorts)
  
  // Configure Engineering VLAN (ports 15-34 untagged)
  const engPorts = {}
  for (let i = 15; i <= 34; i++) {
    engPorts[i] = 2 // Untagged
  }
  await HPAPI.vlans.configurePorts(30, engPorts)
  
  // Configure Guest VLAN (ports 35-44 untagged, 45-48 forbidden)
  const guestPorts = {}
  for (let i = 35; i <= 44; i++) {
    guestPorts[i] = 2 // Untagged
  }
  for (let i = 45; i <= 48; i++) {
    guestPorts[i] = 3 // Forbidden
  }
  await HPAPI.vlans.configurePorts(40, guestPorts)
  
  console.log('✅ Network segmentation configured successfully')
  
  // Verify configuration
  const vlans = await HPAPI.vlans.list()
  console.table(vlans)
}

// Run the setup
await setupNetworkSegmentation()
```

### Automated Monitoring
```javascript
// Comprehensive monitoring script
class SwitchMonitor {
  constructor(switchIP) {
    this.switchIP = switchIP
    this.isRunning = false
  }
  
  async start() {
    await HPAPI.connect(this.switchIP)
    this.isRunning = true
    console.log(`🔍 Started monitoring switch at ${this.switchIP}`)
    
    this.monitorLoop()
  }
  
  stop() {
    this.isRunning = false
    console.log('⏹️ Stopped monitoring')
  }
  
  async monitorLoop() {
    while (this.isRunning) {
      try {
        await this.checkPortStatus()
        await this.checkPortUsage()
        await this.checkStackHealth()
        
        // Wait 60 seconds before next check
        await new Promise(resolve => setTimeout(resolve, 60000))
      } catch (error) {
        console.error('Monitor error:', error.message)
        await new Promise(resolve => setTimeout(resolve, 10000)) // Wait 10s on error
      }
    }
  }
  
  async checkPortStatus() {
    const ports = await HPAPI.ports.list()
    const downPorts = Object.entries(ports)
      .filter(([num, port]) => port.enabled && !port.connected)
      .map(([num]) => num)
    
    if (downPorts.length > 0) {
      console.warn(`⚠️ Enabled ports without link: ${downPorts.join(', ')}`)
    }
  }
  
  async checkPortUsage() {
    const usage = await HPAPI.ports.usage()
    const highUsage = Object.entries(usage)
      .filter(([num, data]) => data.usage > 90)
      .map(([num, data]) => ({ port: num, usage: data.usage }))
    
    if (highUsage.length > 0) {
      console.warn('🚨 High usage ports:', highUsage)
    }
  }
  
  async checkStackHealth() {
    try {
      const members = await HPAPI.stack.members()
      const downMembers = members.filter(m => m.status !== 'Member Up')
      
      if (downMembers.length > 0) {
        console.warn('🚨 Stack members down:', downMembers)
      }
    } catch (error) {
      // Stack might not be configured
    }
  }
}

// Usage
const monitor = new SwitchMonitor('192.168.1.100')
await monitor.start()

// Stop monitoring later
// monitor.stop()
```

### Backup and Restore Configuration
```javascript
// Backup switch configuration
async function backupConfiguration() {
  const config = {
    timestamp: new Date().toISOString(),
    ports: await HPAPI.ports.list(),
    vlans: await HPAPI.vlans.list(),
    stackMembers: await HPAPI.stack.members().catch(() => [])
  }
  
  // Get detailed VLAN port configurations
  config.vlanConfigs = {}
  for (const vlan of config.vlans) {
    config.vlanConfigs[vlan.id] = await HPAPI.vlans.getPorts(vlan.id)
  }
  
  console.log('💾 Configuration backup created')
  console.log(JSON.stringify(config, null, 2))
  
  return config
}

// Restore configuration (be careful!)
async function restoreConfiguration(config) {
  console.log('🔄 Restoring configuration...')
  
  // Restore VLANs
  for (const vlan of config.vlans) {
    if (vlan.id !== '1') { // Don't recreate default VLAN
      try {
        await HPAPI.vlans.create(vlan.id, vlan.name)
        
        // Restore port configuration
        if (config.vlanConfigs[vlan.id]) {
          await HPAPI.vlans.configurePorts(vlan.id, config.vlanConfigs[vlan.id])
        }
      } catch (error) {
        console.warn(`Failed to restore VLAN ${vlan.id}:`, error.message)
      }
    }
  }
  
  // Restore port states
  const enabledPorts = Object.entries(config.ports)
    .filter(([num, port]) => port.enabled)
    .map(([num]) => num)
  
  const disabledPorts = Object.entries(config.ports)
    .filter(([num, port]) => !port.enabled)
    .map(([num]) => num)
  
  if (enabledPorts.length > 0) {
    await HPAPI.ports.enable(enabledPorts)
  }
  
  if (disabledPorts.length > 0) {
    await HPAPI.ports.disable(disabledPorts)
  }
  
  console.log('✅ Configuration restored')
}

// Usage
const backup = await backupConfiguration()
// ... make changes ...
// await restoreConfiguration(backup) // Restore if needed
```

## 🔧 Troubleshooting

### Common Issues

**1. "No switch IP configured" Error**
```javascript
// Always connect first
await HPAPI.connect('192.168.1.100')
```

**2. CORS Errors**
- Use Chrome with disabled security (development only)
- Set up a local proxy server
- Check if switch allows cross-origin requests

**3. "Cannot delete Primary VLAN" Error**
```javascript
// Set another VLAN as primary first
await HPAPI.vlans.setPrimary(100)
// Then delete the old primary
await HPAPI.vlans.delete(1)
```

**4. Port Configuration Not Working**
```javascript
// Check if VLAN exists first
const vlans = await HPAPI.vlans.list()
console.log(vlans)

// Verify port numbers are correct
const ports = await HPAPI.ports.list()
console.log(Object.keys(ports)) // Available ports
```

### Debug Mode
```javascript
// Enable detailed logging
console.log('Switch info:', await HPAPI.utils.info())
console.log('Connection test:', await HPAPI.utils.ping())

// Check all configurations
console.log('Ports:', await HPAPI.ports.list())
console.log('VLANs:', await HPAPI.vlans.list())
console.log('Stack:', await HPAPI.stack.members())
```

## 📚 API Reference Summary

### Connection
- `HPAPI.connect(ip)` - Connect to switch

### Ports
- `HPAPI.ports.list()` - List all ports
- `HPAPI.ports.enable(ports)` - Enable ports
- `HPAPI.ports.disable(ports)` - Disable ports  
- `HPAPI.ports.usage()` - Get usage statistics

### VLANs
- `HPAPI.vlans.list()` - List all VLANs
- `HPAPI.vlans.create(id, name, portConfig)` - Create VLAN
- `HPAPI.vlans.delete(id, force)` - Delete VLAN
- `HPAPI.vlans.getPorts(id)` - Get VLAN port config
- `HPAPI.vlans.configurePorts(id, config)` - Configure ports
- `HPAPI.vlans.setPrimary(id)` - Set Primary VLAN

### Stack
- `HPAPI.stack.members()` - List members
- `HPAPI.stack.candidates()` - List candidates
- `HPAPI.stack.addMember(mac, password)` - Add member
- `HPAPI.stack.removeMember(switchNum)` - Remove member

### Utilities
- `HPAPI.utils.ping()` - Test connection
- `HPAPI.utils.info()` - Get switch info

## 📄 License

This API is provided as-is for managing HP ProCurve switches. Use at your own risk in production environments.

---

**Happy switching! 🚀**
