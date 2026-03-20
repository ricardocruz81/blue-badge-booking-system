# SharePoint List Schema — BlueBadgeBookings

## List Settings
- **List Name:** BlueBadgeBookings
- **List Type:** Custom List
- **Versioning:** Enabled (major versions only)
- **Attachments:** Disabled

## Columns

| Column Name | Internal Name | Type | Required | Default | Notes |
|---|---|---|---|---|---|
| Title | Title | Single line | Yes | | Auto-set by app: BadgeNo + Date |
| BadgeNumber | BadgeNumber | Single line | Yes | | Format: AB1234567 |
| ApplicantName | ApplicantName | Single line | Yes | | Full name |
| ApplicantEmail | ApplicantEmail | Single line | Yes | | For notifications |
| BookingDate | BookingDate | Date and Time | Yes | | Date only (no time) |
| TimeSlot | TimeSlot | Choice | Yes | | 09:00; 10:00; 11:00; 14:00; 15:00; 16:00 |
| ServiceType | ServiceType | Choice | Yes | | Parking Bay; Permit Review; Advisory; General Enquiry |
| BookingStatus | BookingStatus | Choice | Yes | Pending | Pending; Confirmed; Rejected; Cancelled |
| RejectionReason | RejectionReason | Multiple lines | No | | Plain text |
| SubmittedDate | SubmittedDate | Date and Time | No | | Set by app |
| ApprovedBy | ApprovedBy | Single line | No | | Staff name |
| Notes | Notes | Multiple lines | No | | Applicant notes |

## Views

| View Name | Filter | Sort |
|---|---|---|
| All Bookings | None | BookingDate ASC |
| Pending | Status = Pending | BookingDate ASC |
| Today's Bookings | BookingDate = Today | TimeSlot ASC |
| My Bookings | ApplicantEmail = [Me] | BookingDate DESC |
