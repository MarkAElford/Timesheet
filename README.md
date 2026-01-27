# Time Tracker Pro-centia - Timesheet Management System

A modern, user-friendly timesheet entry system designed for teams with drag-and-drop time entry, resizable entries, role-based access control, and comprehensive reporting.

## Features

### Core Functionality
- **Visual Time Entry**: Drag-to-select time slots in 15-minute intervals (8am-7pm, Mon-Fri)
- **Drag to Extend**: Extend time entries by dragging during creation
- **Drag to Reposition**: Move entries to different times/days by dragging them (granular 15-min precision)
- **Resize Entries**: Adjust entry duration by dragging top or bottom edges
- **Smart Overlapping**: Multiple entries per day displayed side-by-side with automatic width adjustment
- **Duration Entry**: Enter hours directly to auto-fill time slots
- **Copy Entries**: Duplicate recurring tasks across days
- **Client Color Coding**: Each client has a unique color for easy visual identification

### User Management
- **Three Role Levels**:
  - **User**: Enter own timesheet, view own entries
  - **Manager**: Enter own timesheet, view team's entries, generate team reports
  - **Admin**: Full system access, user/client/activity management, view all entries

### Smart Features
- **Ticket Learning**: System learns from previous ticket entries
- **Auto-suggestions**: Ticket descriptions auto-populate from history
- **Month Locking**: Automatic locking of previous month's entries (current month + 3 days grace period)
- **Daily Reminders**: Customizable browser notifications (default 5pm)

### Reporting
- **Report Types**:
  - Hours by Client
  - Hours by User
  - Hours by Activity
  - Hours by Ticket
- **CSV Export**: Export all reports to CSV
- **Visual Summaries**: Dashboard with total hours, entries, and percentages

## Quick Start

### Demo Credentials
```
Admin:   username: admin   | password: admin
Manager: username: manager | password: manager
User:    username: user    | password: user
```

### Installation

**Quick Start (Local File - Most Features Work)**

1. Download `timesheet-app.html`

2. Double-click to open in a modern web browser (Chrome, Firefox, Edge)

3. Login with demo credentials

**⚠️ Note:** Browser notifications will NOT work when running from local file (`file://`). All other features work perfectly.

---

**Full Installation (All Features Including Notifications)**

To enable browser notifications, the app must be hosted on a web server:

**Option 1: Python Simple Server (Easiest)**
```bash
# Navigate to the folder containing timesheet-app.html
cd /path/to/folder

# Python 3
python -m http.server 8000

# Python 2
python -m SimpleHTTPServer 8000

# Open browser to: http://localhost:8000/timesheet-app.html
```

**Option 2: Node.js http-server**
```bash
# Install (one time)
npm install -g http-server

# Run in folder with timesheet-app.html
http-server -p 8000

# Open browser to: http://localhost:8000/timesheet-app.html
```

**Option 3: VS Code Live Server Extension**
1. Install "Live Server" extension in VS Code
2. Right-click `timesheet-app.html`
3. Select "Open with Live Server"
4. Browser opens automatically

**Option 4: Web Hosting**
Upload `timesheet-app.html` to any web hosting service:
- GitHub Pages (free)
- Netlify (free)
- Vercel (free)
- Your own web server

**Why Hosting is Required for Notifications:**
Browser notifications are a security-sensitive feature. Browsers block them on `file://` URLs to prevent malicious local HTML files from spamming notifications. When hosted on `http://` or `https://`, the browser allows you to grant permission.

## Usage Guide

### Adding Time Entries

1. Navigate to the **Timesheet** tab
2. Click and drag on time slots to select duration
3. Fill in the entry details:
   - **Client**: Select from dropdown
   - **Activity**: Select activity type
   - **Job Ticket**: Start typing for suggestions
   - **Description**: Enter task description
   - **Duration** (optional): Enter hours to auto-calculate end time
4. Click **Save**

### Moving Time Entries (Drag to Reposition)
1. **Click and hold** on any time entry
2. **Drag** it to a new time slot (any 15-minute slot, same day or different day)
3. **Release** to drop it in the new position
4. The entry keeps its original duration
5. **You can drop on top of existing entries** - they'll arrange side-by-side automatically
6. You can only drag entries to unlocked dates (current month + 3 days)

**Note**: Locked entries (dimmed) cannot be dragged

### Resizing Time Entries
1. **Hover over** any of your time entries
2. **Resize handles appear** at the top and bottom edges (small white bars)
3. **Click and drag** the top handle to adjust start time
4. **Click and drag** the bottom handle to adjust end time  
5. Entries snap to 15-minute intervals
6. You cannot resize below 15 minutes or past another time

**Quick Tip**: This is perfect for fixing entries where you logged wrong times!

### Editing Entries
- Click any existing time entry to edit (if you're not dragging/resizing it)
- Entries can only be edited within current month + 3 days
- Locked entries appear dimmed and cannot be modified

### Overlapping Entries
- Multiple entries at the same time display **side-by-side**
- Each entry automatically adjusts its width to share the space
- Hover over entries to see them in detail
- Click to edit or drag to move
- **Drop entries on top of each other** - they'll automatically arrange themselves

### Copying Entries
1. Click an entry to open it
2. Click **Copy Entry**
3. Select new time slots for the duplicated entry

### Managing Users (Admin Only)
1. Go to **Admin** tab
2. Click **Add User**
3. Fill in user details:
   - Username, Full Name, Email
   - Role (User/Manager/Admin)
   - For Managers: Select team members
4. Set initial password

### Generating Reports
1. Go to **Reports** tab
2. Select:
   - Report Type
   - Date Range
   - User Filter (Managers only)
3. Click **Generate Report**
4. Export to CSV if needed

### Settings
- **Daily Reminders**: Enable/disable and set reminder time
- **Change Password**: Update your password

## Database Migration Guide

The current implementation uses **localStorage** as a flat-file substitute. All database operations are clearly marked with `DATABASE HOOK` comments for easy migration.

### Current Architecture

```javascript
class DataStore {
    // All methods include DATABASE HOOK comments
    getData()           // → SELECT * queries
    saveData()          // → INSERT/UPDATE queries
    getEntries()        // → SELECT with filters
    addEntry()          // → INSERT query
    updateEntry()       // → UPDATE query
    deleteEntry()       // → DELETE query
}
```

### Migration Steps

#### 1. Database Schema

```sql
-- Users Table
CREATE TABLE users (
    id INT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL, -- Use hashing in production
    full_name VARCHAR(100) NOT NULL,
    email VARCHAR(100) NOT NULL,
    role ENUM('user', 'manager', 'admin') NOT NULL,
    team JSON, -- Array of user IDs for managers
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Clients Table
CREATE TABLE clients (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Activities Table
CREATE TABLE activities (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(50) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Time Entries Table
CREATE TABLE time_entries (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    user_id INT NOT NULL,
    date DATE NOT NULL,
    start_time TIME NOT NULL,
    end_time TIME NOT NULL,
    client_id INT NOT NULL,
    activity_id INT NOT NULL,
    ticket VARCHAR(50),
    description TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (client_id) REFERENCES clients(id),
    FOREIGN KEY (activity_id) REFERENCES activities(id),
    INDEX idx_user_date (user_id, date),
    INDEX idx_date_range (date),
    INDEX idx_client (client_id),
    INDEX idx_ticket (ticket)
);

-- Tickets Table (for autocomplete)
CREATE TABLE tickets (
    id INT PRIMARY KEY AUTO_INCREMENT,
    number VARCHAR(50) UNIQUE NOT NULL,
    description TEXT,
    client_id INT,
    last_used TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (client_id) REFERENCES clients(id),
    INDEX idx_last_used (last_used)
);

-- Settings Table
CREATE TABLE user_settings (
    user_id INT PRIMARY KEY,
    enable_reminders BOOLEAN DEFAULT TRUE,
    reminder_time TIME DEFAULT '17:00:00',
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);
```

#### 2. API Endpoints Needed

```javascript
// Authentication
POST   /api/auth/login
POST   /api/auth/logout
POST   /api/auth/change-password

// Time Entries
GET    /api/entries?userId=&startDate=&endDate=&clientId=
POST   /api/entries
PUT    /api/entries/:id
DELETE /api/entries/:id

// Users (Admin only)
GET    /api/users
POST   /api/users
PUT    /api/users/:id
DELETE /api/users/:id

// Clients (Admin only)
GET    /api/clients
POST   /api/clients
DELETE /api/clients/:id

// Activities (Admin only)
GET    /api/activities
POST   /api/activities
DELETE /api/activities/:id

// Tickets
GET    /api/tickets?search=

// Settings
GET    /api/settings/:userId
PUT    /api/settings/:userId

// Reports
GET    /api/reports?type=&startDate=&endDate=&userId=
```

#### 3. Replace DataStore Methods

Find all methods marked with `// DATABASE HOOK:` and replace with API calls:

**Example: getEntries()**

```javascript
// CURRENT (localStorage)
getEntries(filters = {}) {
    const data = this.getData();
    let entries = data.entries;
    // ... filtering logic
    return entries;
}

// REPLACE WITH (API call)
async getEntries(filters = {}) {
    const params = new URLSearchParams(filters);
    const response = await fetch(`/api/entries?${params}`, {
        headers: {
            'Authorization': `Bearer ${sessionStorage.getItem('authToken')}`
        }
    });
    if (!response.ok) throw new Error('Failed to fetch entries');
    return await response.json();
}
```

**Example: addEntry()**

```javascript
// CURRENT (localStorage)
addEntry(entry) {
    const data = this.getData();
    entry.id = Date.now();
    entry.createdAt = new Date().toISOString();
    data.entries.push(entry);
    this.saveData(data);
    return entry;
}

// REPLACE WITH (API call)
async addEntry(entry) {
    const response = await fetch('/api/entries', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json',
            'Authorization': `Bearer ${sessionStorage.getItem('authToken')}`
        },
        body: JSON.stringify(entry)
    });
    if (!response.ok) throw new Error('Failed to add entry');
    return await response.json();
}
```

#### 4. Update Function Calls

Add `async/await` to all functions that call DataStore methods:

```javascript
// BEFORE
function renderTimesheet() {
    const entries = store.getEntries({ startDate, endDate });
    // ... render logic
}

// AFTER
async function renderTimesheet() {
    const entries = await store.getEntries({ startDate, endDate });
    // ... render logic
}
```

#### 5. Add Loading States

```javascript
async function renderTimesheet() {
    showLoadingSpinner();
    try {
        const entries = await store.getEntries({ startDate, endDate });
        // ... render logic
    } catch (error) {
        showToast('Error', 'Failed to load timesheet', 'danger');
    } finally {
        hideLoadingSpinner();
    }
}
```

### Integration with Azure DevOps (Future)

For ADO ticket integration, add these endpoints:

```javascript
// Fetch tickets from Azure DevOps
GET /api/ado/tickets?search=
GET /api/ado/tickets/:id

// Sync ticket data
POST /api/ado/sync
```

Update the ticket autocomplete to fetch from ADO:

```javascript
async function searchADOTickets(search) {
    const response = await fetch(`/api/ado/tickets?search=${search}`);
    const tickets = await response.json();
    return tickets.map(t => ({
        number: t.id,
        description: t.title,
        url: t.url
    }));
}
```

## Browser Compatibility

- Chrome/Edge: Full support
- Firefox: Full support
- Safari: Full support
- Requires modern browser with ES6+ support

## Security Considerations

### Current Implementation (Demo)
- Plain text passwords (localStorage)
- No encryption
- Client-side only validation

### Production Requirements
- **Password Hashing**: Use bcrypt or argon2
- **HTTPS Only**: Enforce SSL/TLS
- **CSRF Protection**: Implement tokens
- **XSS Prevention**: Sanitize inputs
- **Session Management**: Secure JWT tokens
- **Rate Limiting**: Prevent brute force
- **Input Validation**: Server-side validation

## File Structure

```
timesheet-app.html          # Main HTML structure and styling
timesheet-app.js            # Application logic and data management
README.md                   # This file
```

## Data Structure

### Entry Object
```javascript
{
    id: number,
    userId: number,
    date: "YYYY-MM-DD",
    startTime: "HH:MM",
    endTime: "HH:MM",
    clientId: number,
    activityId: number,
    ticket: string,
    description: string,
    createdAt: ISO timestamp,
    updatedAt: ISO timestamp
}
```

### User Object
```javascript
{
    id: number,
    username: string,
    password: string,
    fullName: string,
    email: string,
    role: "user" | "manager" | "admin",
    team: number[] // For managers only
}
```

## Customization

### Time Range
Edit these values in `timesheet-app.js`:

```javascript
// Change start/end times (currently 8am-7pm)
for (let hour = 8; hour <= 18; hour++) {
    // Change to your preferred hours
}
```

### Time Intervals
```javascript
// Change interval (currently 15 minutes)
for (let min = 0; min < 60; min += 15) {
    // Change increment value
}
```

### Grace Period
```javascript
// Change grace period (currently 3 days)
const lockDate = new Date(currentYear, currentMonth + 1, 3);
// Change the day number
```

## Known Limitations

1. **Single Browser**: Data stored in localStorage is browser-specific
2. **No Offline Sync**: Changes not synced across devices
3. **Limited Security**: Not suitable for production without backend
4. **No Backups**: Data only in browser storage
5. **No Audit Trail**: No change history tracking

## Future Enhancements

- [ ] Azure DevOps integration
- [ ] Mobile app version
- [ ] Offline mode with sync
- [ ] Email notifications
- [ ] Calendar integration
- [ ] Bulk entry import/export
- [ ] Advanced analytics dashboard
- [ ] Time tracking timer
- [ ] Project budgets and tracking
- [ ] Approval workflows

## Support

For questions or issues:
1. Check this README
2. Review code comments
3. Test with demo credentials
4. Check browser console for errors

## License

This is a prototype/demo application. Adapt as needed for your organization.

## Version History

**v1.0.0** - Initial Release
- Core timesheet functionality
- User management (3 roles)
- Drag-and-drop interface
- Reports and export
- Daily reminders
- Month locking
- Ticket learning system

---

**Built with vanilla JavaScript - No frameworks required!**
