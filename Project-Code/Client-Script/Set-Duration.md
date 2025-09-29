###  OnChange Client Script: Calculate Duration Between Start Date and End Date

This **onChange Client Script** calculates the inclusive duration in **days** between **Start Date** and **End Date** on the Flexi Leave request form.  
The result is immediately written to the **duration** field when **End Date** changes, using native JavaScript date arithmetic and `g_form` APIs for a responsive UX.

---

####  What It Does

- Listens on **End Date** change.
- Exits during form load or if the field is blank (standard guard pattern in ServiceNow client scripts).
- Converts `YYYY-MM-DD` strings into JavaScript `Date` objects.
- Computes the **time difference in milliseconds**, converts it to **days**, and adds 1 to include both endpoints.
- Updates the **duration** field on the form.
- Handles leap years correctly since JavaScript `Date` objects manage calendar arithmetic automatically.

---

####  Client Script Code

```javascript
function onChange(control, oldValue, newValue, isLoading, isTemplate) {
    if (isLoading || newValue === '') {
        return;
    }

    // Read dates from form
    var start = g_form.getValue('start_date');  // Format: YYYY-MM-DD
    var end = newValue;

    // Convert to Date objects
    var startDate = new Date(start);
    var endDate = new Date(end);

    // Milliseconds difference
    var timeDiff = endDate.getTime() - startDate.getTime();

    // Convert to days
    var days = timeDiff / (1000 * 60 * 60 * 24);

    // Inclusive of both start and end
    var totalDays = Math.round(days) + 1;

    // Write to form
    g_form.setValue('duration', totalDays);
}
