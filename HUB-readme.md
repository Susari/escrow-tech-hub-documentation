# HASIP Hub API & Communication Documentation

## Overview
This document describes the HTTP endpoints, MQTT communication, and internal data flow for the HASIP Hub system.

**Firmware Version:** 1.0.2  
**Model:** HUB(15)

---

# 1. HTTP Endpoints (Web Server on Port 80)

## 1.1 GET `/hasip/config`
**Purpose:**  
Displays the configuration page for WiFi setup.

**Description:**
- Shows an HTML form for entering WiFi credentials
- Displays:
  - Hub PID
  - Hub ID
  - AP SSID
  - Model
  - RTC time (if available)
  - Active schedules count
  - Hub state (Active/Inactive)
  - SD card status
  - OTA status
- Provides navigation links to other endpoints

**Response:**  
HTML page with configuration form

---

## 1.2 POST `/hasip/config`
**Purpose:**  
Save WiFi credentials

**Parameters:**
- `ssid` *(required)* — WiFi network name  
- `password` *(optional)* — WiFi password  

**Response:**
```json
{
  "status": "saved",
  "ssid": "your_ssid",
  "hub_id": "A1B2C3D4E5F6",
  "hub_pid": ""
}
```

**Note:** Device restarts after 2 seconds

---

## 1.3 GET `/info`
**Purpose:**  
Retrieve basic hub information

**Response:**
```json
{
  "hub_pid": "",
  "hub_id": "A1B2C3D4E5F6",
  "model": "HUB(15)",
  "timestamp": "2024-01-15 10:30:45",
  "active_schedules": 5,
  "hub_state": true,
  "firmware_version": "1.0.2"
}
```

---

## 1.4 GET `/ap_info`
**Purpose:**  
Retrieve Access Point configuration details

**Response:**
```json
{
  "hub_pid": "",
  "hub_id": "A1B2C3D4E5F6",
  "model": "HUB(15)",
  "firmware_version": "1.0.2",
  "current_ap_ssid": "HASIP HUB123456ABCDEF",
  "ap_password": "enTrancE",
  "ap_ip": "192.168.4.1",
  "reset_endpoint": "/reset_ap_ssid",
  "connected_stations": 2,
  "active_schedules": 5,
  "local_mqtt_auth": "Username: admin, Password: enTrance_devices_@2hub",
  "local_mqtt_port": 1883,
  "rtc_time": "2024-01-15 10:30:45",
  "hub_state": "active",
  "activation_topic": "activation/route/state",
  "running_partition": "ota_0",
  "ota_pending": false,
  "sd_card_available": true
}
```

---

## 1.5 GET `/reset_ap_ssid`
**Purpose:**  
Reset Access Point SSID to a new random value

**Response:**  
HTML page with countdown

**Note:** Device restarts after 3 seconds

---

## 1.6 GET `/get_network/ip`
**Purpose:**  
Get current network IP address

**Response:**
```json
{
  "hub_id": "A1B2C3D4E5F6",
  "hub_pid": "",
  "ip_address": "192.168.1.100",
  "connected": true,
  "ssid": "MyWiFi"
}
```

---

## 1.7 GET `/schedules`
**Purpose:**  
List all schedules in JSON format

**Response:**
```json
{
  "hub_id": "A1B2C3D4E5F6",
  "count": 3,
  "schedules": [
    {
      "task_id": "289988_e4cc",
      "device_id": "200A3725BF58",
      "pid": "hasip20260406002",
      "type": "daily",
      "start_cmd": "switch1/on",
      "end_cmd": "switch1/off",
      "active": false,
      "start_executed": true,
      "end_executed": true,
      "start_time": "2026-04-10 06:51:00",
      "end_time": "2026-04-10 06:53:00"
    }
  ]
}
```

---

## 1.8 GET `/sd/status`
**Purpose:**  
Get SD card status and file listing

**Response:**
```json
{
  "sd_card_available": true,
  "total_mb": 8192.0,
  "used_mb": 1024.5,
  "free_mb": 7167.5,
  "current_file_index": 3,
  "current_file_size_bytes": 524288,
  "total_data_files": 4,
  "data_files": [
    {
      "name": "data_0.json",
      "size_bytes": 1048576
    }
  ]
}
```

---

## 1.9 POST `/sd/sync`
**Purpose:**  
Force sync all data from SD card to cloud

**Response:**
```json
{
  "status": "sync_started",
  "message": "Syncing all data from SD card to cloud"
}
```

---

## 1.10 POST `/sd/clear`
**Purpose:**  
Clear all data files (maintenance)

**Response:**
```json
{
  "status": "success",
  "deleted_files": 4
}
```

---

## 1.11 POST `/save`
**Purpose:**  
Save configuration and restart

**Response:**
```json
{
  "result": "saved",
  "hub_pid": "",
  "hub_id": "A1B2C3D4E5F6",
  "model": "HUB(15)"
}
```

**Note:** Device restarts after 2 seconds

---

# 2. Cloud MQTT (emqx-staging.hasip.build)

## 2.1 Broker Configuration
| Parameter | Value |
|-----------|-------|
| **Server** | `emqx-staging.hasip.build` |
| **Port** | `1883` |
| **Username** | `firmware_user` |
| **Password** | `Password,123` |

---

## 2.2 Cloud Topics Summary

| Topic | Direction | Purpose |
|-------|-----------|---------|
| `hub/data` | Hub → Cloud | Generic device data |
| `hub/data/{device_type}` | Hub → Cloud | Device type-specific data |
| `hub/commands` | Cloud → Hub | Commands to devices |
| `hub/schedules/task` | Cloud → Hub | Create/update schedule |
| `hub/schedules/task/active` | Both | Manage active schedules |
| `hub/firmware/ota/update` | Cloud → Hub | OTA update commands |
| `activation/route/state` | Both | Hub activation state |
| `hub/status` | Hub → Cloud | Hub status updates |

---

## 2.3 Hub Commands (`hub/commands`)

### Command Format
```json
{
  "hub_id": "DEVICE_ID",
  "device_id": "switch1",
  "pid": "03933393",
  "command": "switch1/on"
}
```
```json
{
  "hub_id": "DEVICE_ID",
  "device_id": "switch1",
  "pid": "03933393",
  "command": "switch2/on",
}
```
```json
{
  "hub_id": "DEVICE_ID",
  "device_id": "switch1",
  "pid": "03933393",
  "command": "switch3/off",
}
```


```json
{
  "hub_id": "DEVICE_ID",
  "device_id": "switch1",
  "pid": "03933393",
  "command": "all/off",
}
```

### Fields Description

| Field | Required | Description |
|-------|----------|-------------|
| `hub_id` | Yes | Target hub identifier |
| `pid` | Yes | Product ID of the device |
| `device_id` | Yes | Device identifier |
| `command` | Yes | Command string |



### Dynamic Device Type
If payload contains `device_type`, hub publishes to `hub/data/{device_type}`:

example `hub/data/single`:

---

## 2.4 Hub Status (`hub/status`)

**Direction:** Hub → Cloud  
**Frequency:** Every 5 minutes

```json
{
  "hub_id": "A1B2C3D4E5F6",
  "hub_pid": "",
  "status": "online",
  "active_schedules": 3
}
```

---

## 2.5 Hub Activation (`activation/route/state`) and Restriction

### Activate/Deactivate Hub (Cloud → Hub)
```json
{
  "hub_id": "A1B2C3D4E5F6",
  "pid": "",
  "changeStatus": true
}
```
```json
{
  "hub_id": "A1B2C3D4E5F6",
  "pid": "",
  "changeStatus": false
}
```

### Status Response (Hub → Cloud)
```json
{
  "hub_id": "A1B2C3D4E5F6",
  "hub_pid": "",
  "status": true
}
```

# 2.6 Physical Button Disabling 
- hub/btnblocker/control
- every device comes with the button disabled so this need to set false on first use

## Cloud
### Disable physical button restriction (hub/btnblocker/control)
```
{
  "hub_id": "1234567890",
  "pid": "",
  "device_pid": "pid-234567890",
  "device_id": "123456789045",
  "btnBlocker": false
}
```

### Enable physical button restriction (hub/btnblocker/control)
```
{
  "hub_id": "1234567890",
  "pid": "",
  "device_pid": "pid-234567890",
  "device_id": "123456789045",
  "btnBlocker": true
}
```

# 2.7 Subscription Button Deactivation 
- hub/account/subscription
- hub/account/subscription/response

## Cloud
### Disable physical button restriction upon subscription expiration (hub/account/subscription)
```
{
  "hub_id": "ABC123",
  "pid": "hub_pid",
  "expiration_date": "2025-12-31",
  "subscription_active": true,
  "devices": [
    {
      "pid": "device_pid_1",
      "device_id": "device_id_1"
    },
    {
      "pid": "device_pid_2",
      "device_id": "device_id_2"
    }
  ]
}
```
### Response (hub/account/subscription/response)
```
{
  "hub_id": "D085849E139C",
  "hub_pid": "",
  "status": "success",
  "expiration_date": "2026-09-20",
  "subscription_active": false,
  "device_count": 1,
  "is_expired": true,
  "timestamp": "2026-09-21 16:48:48"
}
```



# 3. Cloud Schedule Management

## 3.1 Create Schedule (`hub/schedules/task`)

**Direction:** Cloud → Hub

```json
{
  "hub_id": "30415B124B00",
  "pid": "PID-K8ARJ5H8M9",
  "device_id": "381F6F23B3E4",
  "start_date": "2026-07-29 16:51:00",
  "end_date": "2026-07-29 16:52:00",
  "recurrent_type": "daily",
  "start_cmd": "switch1/off",
  "end_cmd": "switch1/on"
}
```

### Recurrence Types
| Type | Description |
|------|-------------|
| `one_time` | Executes once between start and end |
| `daily` | Executes daily at specified times |
| `monthly` | Executes monthly on same day |

## 3.2 Delete Schedule (`hub/schedules/task/active`)

**Direction:** Cloud → Hub

---
```json
{
  "hub_id": "A1B2C3D4E5F6",
  "action": "delete",
  "task_id": "358252_91d2"
}
```


---

## 3.3 Get Active Schedules (`hub/schedules/task/active`)

**Direction:** Cloud → Hub

```json
{
  "hub_id": "A1B2C3D4E5F6",
  "action": "get_active"
}
```

### Response
```json
{
  "hub_id": "A1B2C3D4E5F6",
  "count": 3,
  "schedules": [
    {
      "task_id": "289988_e4cc",
      "device_id": "200A3725BF58",
      "pid": "hasip20260406002",
      "type": "daily",
      "start_cmd": "switch1/on",
      "end_cmd": "switch1/off",
      "active": false,
      "start_executed": true,
      "end_executed": true,
      "start_time": "2026-04-10 06:51:00",
      "end_time": "2026-04-10 06:53:00"
    },
    {
      "task_id": "640547_d527",
      "device_id": "200A3725BF58",
      "pid": "hasip20260406002",
      "type": "one_time",
      "start_cmd": "switch1/on",
      "end_cmd": "switch1/off",
      "active": false,
      "start_executed": true,
      "end_executed": true,
      "start_time": "2026-04-10 07:00:00",
      "end_time": "2026-04-10 07:03:00"
    },
    {
      "task_id": "205130_36dc",
      "device_id": "200A3725BF58",
      "pid": "hasip20260406002",
      "type": "monthly",
      "start_cmd": "switch1/on",
      "end_cmd": "switch1/off",
      "active": true,
      "start_executed": false,
      "end_executed": false,
      "start_time": "2026-04-10 11:00:00",
      "end_time": "2026-04-14 11:03:00"
    }
  ]
}
```

### Get Specific Schedule
```json
{
  "hub_id": "A1B2C3D4E5F6",
  "action": "get_active",
  "task_id": "205130_36dc"
}
```

### Response
```json
{
  "hub_id": "A1B2C3D4E5F6",
  "action": "schedule_data",
  "request_id": "",
  "timestamp": "2026-04-10 19:08:34",
  "schedule": {
    "task_id": "205130_36dc",
    "pid": "hasip20260406002",
    "device_id": "200A3725BF58",
    "recurrent_type": "monthly",
    "start_cmd": "switch1/on",
    "end_cmd": "switch1/off",
    "active": true,
    "start_executed": false,
    "end_executed": false,
    "start_date": "2026-04-10 11:00:00",
    "end_date": "2026-04-14 11:03:00"
  }
}
```

---

# 4. Local MQTT Broker

## 4.1 Broker Configuration
| Parameter | Value |
|-----------|-------|
| **Host** | `{DEVICE_IP}` or `192.168.4.1` (AP mode) |
| **Port** | `1883` |
| **Username** | `admin` |
| **Password** | `enTrance_devices_hubu` |

---

## 4.2 Local Topics Summary

| Topic | Direction | Purpose |
|-------|-----------|---------|
| `hub/schedules/add` | Client → Hub | Create schedule |
| `hub/schedules/delete` | Client → Hub | Delete schedule |
| `hub/schedules/get` | Client → Hub | Get schedules |
| `hub/schedules/response` | Hub → Client | Schedule operation response |
| `hasip/{pid}/{device_id}/cmd` | Hub → Client | Device commands |
| `hasip/#` | Client → Hub | Device data |

---

## 4.3 Create Schedule (`hub/schedules/add`)

**Direction:** Client → Hub

```json
{
  "hub_id": "A1B2C3D4E5F6",
  "pid": "hasip20260406002",
  "device_id": "200A3725BF58",
  "start_date": "2026-05-15 08:00:00",
  "end_date": "2026-05-15 18:00:00",
  "recurrent_type": "daily",
  "start_cmd": "off",
  "end_cmd": "on"
}
```

### Response (`hub/schedules/response`)
```json
{
  "hub_id": "A1B2C3D4E5F6",
  "action": "add_response",
  "status": "success",
  "task_id": "444759_fe06",
  "timestamp": "2026-04-10 18:59:34",
  "schedule": {
    "task_id": "444759_fe06",
    "device_id": "200A3725BF58",
    "pid": "hasip20260406002",
    "recurrent_type": "daily",
    "start_date": "2026-05-15 08:00:00",
    "end_date": "2026-05-15 18:00:00",
    "start_cmd": "ON",
    "end_cmd": "OFF"
  }
}
```

---

## 4.4 Delete Schedule (`hub/schedules/delete`)

**Direction:** Client → Hub

```json
{
  "hub_id": "A1B2C3D4E5F6",
  "action": "delete",
  "task_id": "358252_91d2"
}
```

### Response (`hub/schedules/response`)
```json
{
  "hub_id": "A1B2C3D4E5F6",
  "action": "delete_response",
  "task_id": "289988_e4cc",
  "status": "deleted",
  "timestamp": "2026-04-10 19:03:24"
}
```

---

## 4.5 Get Schedules (`hub/schedules/get`)

**Direction:** Client → Hub

### Get All Active Schedules
```json
{
  "hub_id": "A1B2C3D4E5F6",
  "action": "get_active"
}
```

### Get All Schedules (including inactive)
```json
{
  "hub_id": "A1B2C3D4E5F6",
  "action": "get_active",
  "include_inactive": true
}
```

### Get Specific Schedule
```json
{
  "hub_id": "A1B2C3D4E5F6",
  "action": "get_active",
  "task_id": "205130_36dc"
}
```

### Response (Specific Schedule)
```json
{
  "hub_id": "A1B2C3D4E5F6",
  "action": "schedule_data",
  "request_id": "",
  "timestamp": "2026-04-10 19:08:34",
  "schedule": {
    "task_id": "205130_36dc",
    "pid": "hasip20260406002",
    "device_id": "200A3725BF58",
    "recurrent_type": "monthly",
    "start_cmd": "switch1/on",
    "end_cmd": "switch1/off",
    "active": true,
    "start_executed": false,
    "end_executed": false,
    "start_date": "2026-04-10 11:00:00",
    "end_date": "2026-04-14 11:03:00"
  }
}
```

### Response (All Schedules)
```json
{
  "hub_id": "A1B2C3D4E5F6",
  "count": 3,
  "schedules": [
    {
      "task_id": "289988_e4cc",
      "device_id": "200A3725BF58",
      "pid": "hasip20260406002",
      "type": "daily",
      "start_cmd": "switch1/on",
      "end_cmd": "switch1/off",
      "active": false,
      "start_executed": true,
      "end_executed": true,
      "start_time": "2026-04-10 06:51:00",
      "end_time": "2026-04-10 06:53:00"
    }
  ]
}
```

---

## 4.6 Device Commands (`hasip/{pid}/{device_id}/cmd`)

**Direction:** Hub → Client (published by hub)

### Command Format
```json
{
  "command": "switch1/off"
}
```
---

## 4.7 Device Data (`hasip/#`)

**Direction:** Client → Hub

### Data Format
```json
{
  "device_id": "SENSOR_001",
  "device_type": "temperature_sensor",
  "value": 24.5,
  "unit": "C"
}
```

---

# 5. OTA Updates

## 5.1 OTA Commands (`hub/firmware/ota/update`)

**Direction:** Cloud → Hub

### Commands Summary

| Command | Description | Required Fields |
|---------|-------------|-----------------|
| `update` | Start firmware update | `firmware_url` |
| `status` | Get update status | None |
| `rollback` | Rollback to previous | None |
| `version` | Get firmware version | None |

### Initiate Update
```json
{
  "hub_id": "A1B2C3D4E5F6",
  "pid": "",
  "ota_command": "update",
  "firmware_url": "https://example.com/firmware.bin",
  "required_version": "1.0.2"
}
```

### Broadcast Update (all hubs)
```json
{
  "ota_command": "update",
  "firmware_url": "https://example.com/firmware.bin"
}
```

### Get Status
```json
{
  "hub_id": "A1B2C3D4E5F6",
  "pid": "",
  "ota_command": "status"
}
```

### Initiate Rollback
```json
{
  "hub_id": "A1B2C3D4E5F6",
  "pid": "",
  "ota_command": "rollback"
}
```

### Get Version
```json
{
  "hub_id": "A1B2C3D4E5F6",
  "pid": "",
  "ota_command": "version"
}
```

---

## 5.2 OTA Responses

### Update Started
```json
{
  "hub_id": "A1B2C3D4E5F6",
  "hub_pid": "",
  "ota_status": "started",
  "message": "OTA update initiated",
  "firmware_url": "https://example.com/firmware.bin",
  "is_broadcast": false,
  "timestamp": "2024-01-15 10:30:45"
}
```

### Downloading Progress
```json
{
  "hub_id": "A1B2C3D4E5F6",
  "hub_pid": "",
  "ota_status": "downloading",
  "progress_percent": 45,
  "bytes_written": 524288,
  "total_bytes": 1048576,
  "is_broadcast": false,
  "timestamp": "2024-01-15 10:31:00"
}
```

### Update Success
```json
{
  "hub_id": "A1B2C3D4E5F6",
  "hub_pid": "",
  "ota_status": "success",
  "message": "Update completed, rebooting...",
  "is_broadcast": false,
  "timestamp": "2024-01-15 10:32:00"
}
```

### Version Info
```json
{
  "hub_id": "A1B2C3D4E5F6",
  "hub_pid": "",
  "firmware_version": "1.0.2",
  "ota_status": "version_info",
  "running_partition": "ota_0",
  "timestamp": "2024-01-15 10:30:45"
}
```
---

## 5.3 OTA Process

1. **Command Received** → Validation and authentication
2. **Firmware Download** → HTTP GET with headers:
   - `X-Hub-ID: {hub_id}`
   - `X-Hub-PID: {pid}`
   - `X-Hub-Model: HUB(15)`
   - `X-Broadcast: true/false`
3. **Progressive Write** → 2KB buffer, progress reporting
4. **Verification** → Integrity check
5. **Boot Partition Set** → Set new partition as active
6. **Reboot** → Device restarts with new firmware
7. **Validation** → 30-second grace period
8. **Confirmation** → Mark as valid or rollback

---

# 6. SD Card Data Storage

## 6.1 File Structure
```
/data_0.json
/data_1.json
/data_2.json
...
```

## 6.2 Data Format (JSON Lines)
Each line is a JSON object:
```json
{"topic":"hub/data/temperature_sensor","payload":"{\"device_id\":\"SENSOR_001\",\"temperature\":24.5}","timestamp":"2024-01-15 10:30:45","hub_id":"A1B2C3D4E5F6","hub_pid":""}
```

## 6.3 Features
- Automatic file rotation at 1MB
- Sequential file numbering
- JSON line format for easy parsing
- Automatic cloud sync when connected
- Retry mechanism for failed messages

## 6.4 Sync Process
1. **Trigger:** Cloud connection established
2. **Process:**
   - Scan all `data_*.json` files
   - Read messages line by line
   - Publish to cloud
   - Remove successful messages
   - Keep failed messages for retry
3. **Interval:** Every 5 seconds when connected
4. **Retry:** Failed messages preserved in file

---

# 7. Data Flow Diagrams

## 7.1 Local Device → Cloud Flow
```
┌──────────────┐
│ Local Device │
│ (MQTT)       │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│  Local MQTT  │
│  Broker      │
│ (Port 1883)  │
└──────┬───────┘
       │
       ▼
┌──────────────┐     ┌──────────────┐
│ Hub Firmware │────▶│ SD Card      │
│ (Process)    │     │ (Backup)     │
└──────┬───────┘     └──────────────┘
       │
       ▼
┌──────────────┐
│ Cloud MQTT   │
│ (emqx-staging│
│ .hasip.build)│
└──────────────┘
```

## 7.2 Schedule Execution Flow
```
┌──────────────┐
│ Cloud/Local  │
│ Schedule API │
└──────┬───────┘
       │
       ▼
┌──────────────┐     ┌──────────────┐
│ Hub Schedules│────▶│ Flash Storage│
│ Manager      │     │ (Preferences)│
└──────┬───────┘     └──────────────┘
       │
       ▼
┌──────────────┐
│ RTC Time     │
│ Check        │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ Execute      │
│ Command      │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ Local MQTT   │
│ Command      │
└──────────────┘
```

---

# 8. Error Codes & Handling

## 8.1 HTTP Status Codes
| Code | Meaning |
|------|---------|
| 200 | Success |
| 400 | Bad Request (missing parameters) |
| 404 | Not Found |

## 8.2 MQTT Connection States
| State | Description |
|-------|-------------|
| `connected` | Successfully connected to broker |
| `disconnected` | Not connected, attempting reconnect |
| `failed` | Connection attempt failed |

## 8.3 Schedule Error Responses
| Error | Description |
|-------|-------------|
| `Missing start_date or end_date` | Required fields missing |
| `Invalid date format` | Use YYYY-MM-DD HH:MM:SS |
| `Schedule not found` | Task ID doesn't exist |
| `Wrong hub_id` | Command intended for different hub |

---

# 9. Best Practices

## 9.1 MQTT Best Practices
- Always include `hub_id` in cloud commands
- Use QoS 1 for critical commands
- Subscribe to response topics for confirmation
- Implement reconnection logic in clients

## 9.2 Schedule Best Practices
- Validate date formats before sending
- Use appropriate recurrence types
- Set realistic time ranges
- Monitor schedule execution status

## 9.3 OTA Best Practices
- Verify firmware size before update
- Use stable internet connection
- Keep backup firmware accessible
- Test on staging devices first

## 9.4 SD Card Best Practices
- Use Class 10 or higher SD cards
- Format as FAT32 with 4KB clusters
- Maintain at least 10MB free space
- Regularly monitor sync status

---

# 10. Quick Reference

## 10.1 Connection Details
```
HTTP Interface:
  - URL: http://{DEVICE_IP}/
  - AP Mode: http://192.168.4.1/

Local MQTT:
  - Host: {DEVICE_IP}
  - Port: 1883
  - Username: admin
  - Password: enTrance_devices_hubu

Cloud MQTT:
  - Host: emqx-staging.hasip.build
  - Port: 1883
  - Username: firmware_user
  - Password: Password,123
```

## 10.2 Key Topics
```
Local:
  - hub/schedules/add
  - hub/schedules/delete
  - hub/schedules/get
  - hub/schedules/response
  - hasip/{pid}/{device_id}/cmd

Cloud:
  - hub/data
  - hub/commands
  - hub/schedules/task
  - hub/schedules/task/active
  - hub/firmware/ota/update
  - activation/route/state
  - hub/status
```

## 10.3 Key Endpoints
```
GET  /hasip/config
POST /hasip/config
GET  /info
GET  /ap_info
GET  /reset_ap_ssid
GET  /get_network/ip
GET  /schedules
GET  /sd/status
POST /sd/sync
POST /sd/clear
POST /save
```
