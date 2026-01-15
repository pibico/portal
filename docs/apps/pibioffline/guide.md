# pibiOffline Guide

**PWA for Work Attendance**

A Progressive Web App (PWA) for QR code-based attendance tracking with offline capabilities, designed for organizations managing field employees and remote workers.

## Table of Contents

- [Features](#features)
- [System Architecture](#system-architecture)
- [Installation](#installation)
- [Setup](#setup)
- [Configuration](#configuration)
- [Usage](#usage)
- [Offline Functionality](#offline-functionality)
- [DocTypes](#doctypes)
- [API Reference](#api-reference)
- [Security & Permissions](#security--permissions)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)

## Features

### Core Features
- **QR Code Scanning**: High-performance camera-based QR code scanning
- **Offline-First Design**: Full functionality without internet connection
- **Progressive Web App**: Installable on mobile devices as native app
- **Automatic Sync**: Background synchronization when connection restored
- **Geolocation Tracking**: Optional GPS location capture for attendance
- **Photo Verification**: Optional photo requirement for check-ins
- **Multi-Organization Support**: Manage employees across different organizations

### Advanced Features
- **Hybrid QR Detection**: Client-side jsQR with server-side OpenCV fallback
- **Smart Caching**: Automatic caching of accessible employee data
- **Session Management**: Tracks work sessions with check-in/out pairs
- **Permission-Based Access**: Role-based visibility of employee data
- **Real-time Status**: Visual indicators for online/offline status
- **Batch Sync**: Efficient synchronization of multiple offline records

## System Architecture

### Frontend Components
- **Custom HTML Block**: Isolated workspace implementation
- **IndexedDB Storage**: Local database for offline data
- **Service Worker**: PWA functionality and caching
- **WebRTC Camera**: Direct camera access for QR scanning

### Backend Components
- **Frappe Framework**: Core application platform
- **Custom DocTypes**: Employee, attendance, and session management
- **REST APIs**: Attendance sync and data retrieval
- **OpenCV Integration**: Advanced QR code processing

### Data Flow
1. **Check-in Process**:
   - QR scan → Employee validation → Location/Photo capture → Local storage → Server sync
2. **Check-out Process**:
   - QR scan → Session lookup → Location capture → Session completion → Server sync
3. **Offline Queue**:
   - Failed syncs → IndexedDB queue → Retry on connection → Update status

## Installation

```bash
# Get the app
bench get-app https://github.com/pibi/pibiOffline

# Install on your site
bench --site your-site.com install-app pibioffline

# Run migrations
bench --site your-site.com migrate

# Clear cache
bench --site your-site.com clear-cache

# Optional: Install OpenCV for enhanced QR detection
pip install opencv-python numpy
```

## Setup

### 1. Create Master Data

#### PL Person (Employee Personal Info)
```
1. Go to PL Person list
2. Create records with:
   - Full Name
   - User (optional - links to system user)
   - Other personal details
```

#### PL Organization
```
1. Go to PL Organization list
2. Create organization records with:
   - Organization Name
   - Organization Code
   - Address/contact details
```

#### PL Employee Assignment
```
1. Go to PL Employee Assignment list
2. Create assignments linking person to organization:
   - Person (link to PL Person)
   - Organization (link to PL Organization)
   - Employee Code (unique identifier)
   - QR Code (auto-generated or manual)
   - Status (Active/Inactive)
   - Location Required (checkbox)
   - Picture Required (checkbox)
   - User (optional - for direct user access)
   - Reporting Manager (optional - for hierarchy)
```

### 2. Configure Workspace

```
1. Go to Workspace List
2. Create/edit "pibiOffline" workspace
3. Add Custom HTML Block named "PL pibiOffline"
4. Configure block with provided HTML/JS/CSS
```

### 3. Set Up Permissions

The app automatically handles permissions based on:
- **System Manager/HR Manager**: Access to all employees
- **Regular Users**: Access to:
  - Their own assignments (via user field)
  - Employees in their organizations
  - Employees they manage

## Configuration

### Employee Assignment Settings

```python
# Location requirement for check-in
location_required = 1  # Enforces GPS capture

# Photo requirement for check-in  
picture_required = 1   # Enforces photo capture

# Note: Requirements only apply to check-in, not check-out
```

### Offline Cache Settings

```javascript
// Auto-cache interval (default: 30 minutes)
const CACHE_INTERVAL = 30 * 60 * 1000;

// Cached data includes:
// - Employee assignments accessible to user
// - Organization details
// - Person names
// - QR codes and requirements
```

## Usage

### For Employees

1. **First Time Setup**:
   - Open pibiOffline workspace while online
   - Grant camera and location permissions when prompted
   - Install as PWA if desired (Add to Home Screen)

2. **Daily Check-in**:
   - Open pibiOffline app
   - Click "Scan QR Code"
   - Scan your employee QR code
   - Take photo if required
   - Confirm location if required
   - View confirmation message

3. **Daily Check-out**:
   - Scan QR code again
   - System automatically detects check-out
   - View work duration
   - Data syncs when online

### For Administrators

1. **Monitor Attendance**:
   - View PL Resource Attendance list
   - Filter by date, employee, organization
   - Check sync status for offline records

2. **Manage Employees**:
   - Update PL Employee Assignment status
   - Configure location/photo requirements
   - Assign reporting managers
   - Link to user accounts

3. **View Work Sessions**:
   - Access PL Work Session records
   - See complete check-in/out pairs
   - Track work duration and locations

## Offline Functionality

### How It Works

1. **Initial Cache**:
   - On workspace load, caches accessible employees
   - Stores in IndexedDB with indexes for fast lookup
   - Updates every 30 minutes when online

2. **Offline Detection**:
   - Visual indicator shows connection status
   - All features remain functional
   - Data queued for sync

3. **Data Storage**:
   ```javascript
   // IndexedDB structure
   {
     attendance: {
       id: auto-increment,
       qrCode: string,
       type: 'check-in' | 'check-out',
       timestamp: ISO string,
       location: string,
       latitude: number,
       longitude: number,
       photo: base64 string,
       synced: boolean,
       localId: unique string
     },
     resources: {
       name: docname,
       qr_code: string (indexed),
       person_name: string,
       organization_name: string,
       location_required: boolean,
       picture_required: boolean
     }
   }
   ```

4. **Sync Process**:
   - Automatic on connection restore
   - Manual sync button available
   - Handles duplicates via localId
   - Updates UI on completion

## DocTypes

### PL Person
- **Purpose**: Store employee personal information
- **Key Fields**: full_name, user, contact details
- **Links**: PL Employee Assignment

### PL Organization  
- **Purpose**: Define organizational units
- **Key Fields**: organization_name, code, address
- **Links**: PL Employee Assignment

### PL Employee Assignment
- **Purpose**: Link persons to organizations with work details
- **Key Fields**:
  - person (Link to PL Person)
  - organization (Link to PL Organization)
  - employee_code (unique)
  - qr_code (for scanning)
  - status (Active/Inactive)
  - location_required (Check field)
  - picture_required (Check field)
  - user (Link to User - optional)
  - reporting_manager (Link - optional)

### PL Resource Attendance
- **Purpose**: Store individual attendance events
- **Key Fields**:
  - resource (Link to PL Employee Assignment)
  - attendance_type (Check In/Check Out)
  - timestamp (Datetime)
  - location, latitude, longitude
  - photo (Attach Image)
  - synced (Check)
  - local_id (for deduplication)

### PL Work Session
- **Purpose**: Track complete work sessions
- **Key Fields**:
  - employee (Link to PL Employee Assignment)
  - work_date (Date)
  - check_in_time, check_out_time
  - check_in_location, check_out_location
  - status (Active/Completed)
  - Duration calculated automatically

## API Reference

### Attendance APIs

#### sync_attendance
```python
@frappe.whitelist()
def sync_attendance(**kwargs):
    """
    Create/update attendance record
    
    Args:
        qr_code: Employee QR code
        type: 'check-in' or 'check-out'
        time: Timestamp
        location: Address string
        latitude: GPS latitude
        longitude: GPS longitude
        local_id: Client-side unique ID
        photo: Base64 image data
    
    Returns:
        {
            success: boolean,
            attendance_id: string,
            employee_name: string,
            organization_name: string,
            message: string
        }
    """
```

#### get_resource_by_qr
```python
@frappe.whitelist()
def get_resource_by_qr(qr_code):
    """Get employee details by QR code"""
```

#### get_active_work_session
```python
@frappe.whitelist()
def get_active_work_session(qr_code):
    """Get today's active session for employee"""
```

#### get_user_accessible_resources
```python
@frappe.whitelist()
def get_user_accessible_resources():
    """Get all employees accessible to logged-in user"""
```

### Advanced APIs

#### process_qr_with_opencv
```python
@frappe.whitelist()
def process_qr_with_opencv(image_base64):
    """Process difficult QR codes using OpenCV"""
```

## Security & Permissions

### Permission Model
1. **Role-Based Access**:
   - System Manager: Full access
   - HR Manager: Full access
   - Regular Users: Limited by assignment

2. **Data Visibility**:
   - Users see their own assignments
   - Users see colleagues in same organization
   - Managers see their reportees

3. **Security Features**:
   - Server-side validation
   - Duplicate prevention via local_id
   - Photo verification option
   - Location verification option

### Browser Permissions
- **Camera**: Required for QR scanning
- **Location**: Required if location_required is set
- **Storage**: Required for offline functionality

**Important**: Permissions must be granted while online

## Troubleshooting

### Common Issues

1. **"Camera shows black screen"**
   - Solution: Ensure proper cleanup in stopScanning()
   - Clear browser cache and reload

2. **"Employee not active" in offline mode**
   - Solution: Employee not cached, go online to cache
   - Check user has access to employee

3. **"Location/Camera permission denied"**
   - Solution: Grant permissions while online first
   - Check browser settings

4. **"Unknown column 'ea.user' error"**
   - Solution: Run doctype update in UI
   - System has backward compatibility

5. **Duplicate attendance records**
   - Solution: System uses local_id for deduplication
   - Check IndexedDB for corrupted data

### Debugging Tools

```javascript
// Check IndexedDB contents
pibioffline.storage.db

// View cached resources
await pibioffline.storage.getResourceByQR('EMP001')

// Check unsynced records
await pibioffline.storage.getUnsyncedAttendance()

// Force sync
await pibioffline.storage.syncToServer()
```

### Contributing

This app uses `pre-commit` for code formatting and linting. Please [install pre-commit](https://pre-commit.com/#installation) and enable it for this repository:

```bash
cd apps/pibioffline
pre-commit install
```

Pre-commit is configured to use the following tools for checking and formatting your code:

- ruff
- eslint
- prettier
- pyupgrade

### License

mit