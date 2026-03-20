# Blue Badge Booking System (Power Apps)

A Power Apps canvas application for managing Blue Badge holder bookings, with a SharePoint list backend, duplicate prevention, validation logic, and an automated approval workflow via Power Automate.

---

## Architecture

```
┌─────────────────────────┐
│   Power Apps            │
│   Canvas App            │
│                         │
│  ┌─────────────────┐    │
│  │ Booking Form    │    │
│  │ - Date picker   │    │
│  │ - Time slots    │    │
│  │ - Badge number  │    │
│  │ - Validation    │    │
│  └────────┬────────┘    │
└───────────┼─────────────┘
            │ On Submit
            ▼
┌─────────────────────────┐
│   SharePoint List       │
│   "BlueBadgeBookings"   │
│   + Validation check    │
└───────────┬─────────────┘
            │ On Create
            ▼
┌─────────────────────────┐
│   Power Automate        │
│   Approval Flow         │
│   → Approval email      │
│   → Status update       │
│   → Notify applicant    │
└─────────────────────────┘
```

---

## Project Structure

```
4-blue-badge-booking-system
├── README.md
├── power-apps
│   ├── app-screens.md          # Screen-by-screen description
│   ├── formulas.md             # Key Power Fx formulas
│   └── data-connections.md    # Data source setup
├── sharepoint
│   └── booking-list-schema.md # SharePoint list columns
└── power-automate
    └── booking-approval-flow.json
```

---

## SharePoint List: BlueBadgeBookings

| Column | Type | Notes |
|--------|------|-------|
| Title | Single line | Auto-generated: Badge No + Date |
| BadgeNumber | Single line | Required — unique badge reference |
| ApplicantName | Single line | Full name of badge holder |
| ApplicantEmail | Single line | For notifications |
| BookingDate | Date/Time | Date of requested service |
| TimeSlot | Choice | 09:00 / 10:00 / 11:00 / 14:00 / 15:00 |
| ServiceType | Choice | Parking Bay / Permit Review / Advisory |
| BookingStatus | Choice | Pending / Confirmed / Rejected / Cancelled |
| RejectionReason | Multiple lines | If rejected |
| SubmittedDate | Date/Time | Auto-set |
| ApprovedBy | Person | Staff who approved |
| Notes | Multiple lines | Additional info |

---

## Power Apps Screens

### Screen 1: Home
- Welcome banner
- "New Booking" button
- "My Bookings" button (filtered to current user)
- Quick status summary

### Screen 2: New Booking Form
- Badge number input with format validation
- Date picker (no weekends, no past dates)
- Time slot gallery (shows available slots only)
- Service type dropdown
- Notes field
- Submit button with duplicate check

### Screen 3: My Bookings
- Gallery showing all user bookings
- Status badge (colour-coded)
- Cancel button for Pending bookings

### Screen 4: Admin View (Staff only)
- All bookings gallery
- Filter by status, date, service type
- Approve / Reject with comments

---

## Key Power Fx Formulas

### Duplicate Prevention
```powerfx
// Check if slot already booked before submitting
If(
    CountRows(
        Filter(
            BlueBadgeBookings,
            BookingDate = DatePicker1.SelectedDate &&
            TimeSlot = DropdownTimeSlot.Selected.Value &&
            BookingStatus <> "Cancelled"
        )
    ) > 0,
    Notify("This time slot is already booked. Please choose another.", NotificationType.Warning),
    SubmitForm(BookingForm)
)
```

### Date Validation (Weekdays Only)
```powerfx
// Disable submit if weekend selected
If(
    Weekday(DatePicker1.SelectedDate) = 1 ||
    Weekday(DatePicker1.SelectedDate) = 7,
    Notify("Bookings are only available Monday to Friday.", NotificationType.Warning)
)
```

### Status Colour Coding
```powerfx
// Background colour of status label
Switch(
    ThisItem.BookingStatus,
    "Confirmed",  RGBA(34, 139, 34, 1),
    "Pending",    RGBA(255, 165, 0, 1),
    "Rejected",   RGBA(220, 20, 60, 1),
    "Cancelled",  RGBA(128, 128, 128, 1),
    RGBA(0,0,0,1)
)
```

---

## Skills Demonstrated
- Power Apps canvas app design
- Power Fx formula logic
- SharePoint list as a data backend
- Duplicate detection and prevention
- Power Automate approval notifications
- Role-based views (user vs admin)
