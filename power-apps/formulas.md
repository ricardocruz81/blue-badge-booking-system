# Power Apps — Key Formulas and Screen Logic

All Power Fx formulas used in the Blue Badge Booking System canvas app.

---

## App Setup

**Data Sources to connect:**
- SharePoint list: `BlueBadgeBookings` (from your SharePoint site)

**Variables to initialise on App Start:**
```powerfx
// App.OnStart
Set(varCurrentUser, User());
Set(varIsAdmin, 
    User().Email in ["admin1@yourorg.com", "admin2@yourorg.com"]
);
ClearCollect(
    colAvailableSlots,
    ["09:00","10:00","11:00","14:00","15:00","16:00"]
);
```

---

## Screen 1 — Home Screen

### "New Booking" Button — OnSelect
```powerfx
// Reset form variables before navigating
Set(varSelectedDate, Today());
Set(varSelectedSlot, "");
Reset(txtBadgeNumber);
Navigate(scrNewBooking, ScreenTransition.Slide)
```

### "My Bookings" Button — OnSelect
```powerfx
Navigate(scrMyBookings, ScreenTransition.Slide)
```

### Bookings Summary Label — Text
```powerfx
// Show count of user's active bookings
"You have " & 
CountRows(
    Filter(BlueBadgeBookings, 
        SubmittedBy.Email = varCurrentUser.Email,
        BookingStatus <> "Cancelled"
    )
) & " active booking(s)"
```

---

## Screen 2 — New Booking Form

### Date Picker — OnChange
```powerfx
// Clear the time slot selection when date changes
Set(varSelectedSlot, "");
ClearCollect(
    colBookedSlots,
    ShowColumns(
        Filter(BlueBadgeBookings,
            DateValue(Text(BookingDate)) = DatePicker1.SelectedDate,
            BookingStatus <> "Cancelled"
        ),
        "TimeSlot"
    )
);
```

### Date Picker — MinDate (no past dates)
```powerfx
Today()
```

### Date Picker — DisplayMode (disable weekends)
```powerfx
If(
    Weekday(DatePicker1.SelectedDate, StartOfWeek.Monday) > 5,
    DisplayMode.Disabled,
    DisplayMode.Edit
)
```

### Weekend Warning Label — Visible
```powerfx
Weekday(DatePicker1.SelectedDate, StartOfWeek.Monday) > 5
```

### Time Slot Gallery — Items
```powerfx
// Show all slots, mark booked ones
AddColumns(
    colAvailableSlots,
    "IsBooked",
    ThisRecord.Value in colBookedSlots.TimeSlot
)
```

### Time Slot Gallery — Fill (colour coding)
```powerfx
If(
    ThisItem.IsBooked, RGBA(220, 53, 69, 0.2),   // Red tint = booked
    varSelectedSlot = ThisItem.Value, RGBA(40, 167, 69, 0.3), // Green = selected
    RGBA(255, 255, 255, 1)                         // White = available
)
```

### Time Slot Item — OnSelect
```powerfx
If(
    !ThisItem.IsBooked,
    Set(varSelectedSlot, ThisItem.Value),
    Notify("This time slot is already booked. Please choose another.", NotificationType.Warning)
)
```

### Badge Number — Validation (OnChange)
```powerfx
// UK Blue Badge format: 2 letters + up to 7 digits
If(
    Not(IsMatch(txtBadgeNumber.Text, "^[A-Za-z]{2}[0-9]{5,7}$")),
    Set(varBadgeError, "Please enter a valid badge number (e.g. AB1234567)"),
    Set(varBadgeError, "")
)
```

### Submit Button — DisplayMode
```powerfx
// Disable submit unless all fields valid
If(
    IsBlank(txtBadgeNumber.Text) ||
    Len(varBadgeError) > 0 ||
    IsBlank(varSelectedSlot) ||
    Weekday(DatePicker1.SelectedDate, StartOfWeek.Monday) > 5,
    DisplayMode.Disabled,
    DisplayMode.Edit
)
```

### Submit Button — OnSelect
```powerfx
// Final duplicate check before submitting
If(
    CountRows(
        Filter(BlueBadgeBookings,
            BookingDate = DatePicker1.SelectedDate,
            TimeSlot = varSelectedSlot,
            BookingStatus <> "Cancelled"
        )
    ) > 0,
    Notify("This slot was just taken. Please select another time.", NotificationType.Error),

    // Check badge not already booked on same date
    CountRows(
        Filter(BlueBadgeBookings,
            BadgeNumber = txtBadgeNumber.Text,
            DateValue(Text(BookingDate)) = DatePicker1.SelectedDate,
            BookingStatus <> "Cancelled"
        )
    ) > 0,
    Notify("This badge number already has a booking on this date.", NotificationType.Warning),

    // All checks passed — submit
    Patch(
        BlueBadgeBookings,
        Defaults(BlueBadgeBookings),
        {
            Title:          txtBadgeNumber.Text & " - " & Text(DatePicker1.SelectedDate, "dd/mm/yyyy"),
            BadgeNumber:    txtBadgeNumber.Text,
            ApplicantName:  varCurrentUser.FullName,
            ApplicantEmail: varCurrentUser.Email,
            BookingDate:    DatePicker1.SelectedDate,
            TimeSlot:       varSelectedSlot,
            ServiceType:    drpServiceType.Selected.Value,
            BookingStatus:  "Pending",
            SubmittedDate:  Now(),
            Notes:          txtNotes.Text
        }
    );
    Notify("Booking submitted successfully! You will receive a confirmation email.", NotificationType.Success);
    Navigate(scrMyBookings, ScreenTransition.Slide)
)
```

---

## Screen 3 — My Bookings

### Bookings Gallery — Items
```powerfx
Sort(
    Filter(
        BlueBadgeBookings,
        ApplicantEmail = varCurrentUser.Email
    ),
    BookingDate,
    SortOrder.Ascending
)
```

### Status Badge — Fill colour
```powerfx
Switch(
    ThisItem.BookingStatus,
    "Confirmed",  RGBA(40,  167, 69,  1),
    "Pending",    RGBA(255, 193, 7,   1),
    "Rejected",   RGBA(220, 53,  69,  1),
    "Cancelled",  RGBA(108, 117, 125, 1),
    RGBA(0,0,0,1)
)
```

### Cancel Button — Visible
```powerfx
ThisItem.BookingStatus = "Pending" &&
ThisItem.BookingDate >= Today()
```

### Cancel Button — OnSelect
```powerfx
If(
    Confirm("Are you sure you want to cancel this booking?"),
    Patch(
        BlueBadgeBookings,
        ThisItem,
        {BookingStatus: "Cancelled"}
    );
    Notify("Booking cancelled.", NotificationType.Information),
    false
)
```

---

## Screen 4 — Admin View (Staff Only)

### Screen — Visible (restrict access)
```powerfx
varIsAdmin
```

### Admin Gallery — Items
```powerfx
Sort(
    If(
        drpStatusFilter.Selected.Value = "All",
        BlueBadgeBookings,
        Filter(BlueBadgeBookings, BookingStatus = drpStatusFilter.Selected.Value)
    ),
    BookingDate,
    SortOrder.Ascending
)
```

### Approve Button — OnSelect
```powerfx
Patch(
    BlueBadgeBookings,
    galAdminBookings.Selected,
    {
        BookingStatus: "Confirmed",
        ApprovedBy: varCurrentUser.FullName
    }
);
Notify("Booking confirmed.", NotificationType.Success)
```

### Reject Button — OnSelect
```powerfx
If(
    IsBlank(txtRejectionReason.Text),
    Notify("Please enter a rejection reason.", NotificationType.Warning),
    Patch(
        BlueBadgeBookings,
        galAdminBookings.Selected,
        {
            BookingStatus:    "Rejected",
            RejectionReason:  txtRejectionReason.Text,
            ApprovedBy:       varCurrentUser.FullName
        }
    );
    Notify("Booking rejected.", NotificationType.Information)
)
```
