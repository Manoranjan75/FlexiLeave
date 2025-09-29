#  Business Rule: Calculate Leaves (Async)

The **Calculate Leaves** business rule is an **asynchronous (async) server-side rule** in the **FlexiLeave** application.  
It ensures that when a leave request is **submitted and approved**, the corresponding employee’s **leave balance** is automatically updated in the **Leave Bucket** table.

---

##  Configuration Details
- **Table:** `Leave Request (x_872769_flexile_0_leave_request_5)`  
- **Application:** FlexiLeave  
- **Execution:** Runs **Async** after record insertion  
- **Conditions:**
  - Approval = **Approved**  
  - Status = **Submitted**  
- **Trigger:** On **Insert** event  

---

#  How It Works

When an employee submits a leave request:

1. **Status** is set to _Submitted_  
2. **Approval** is granted (_Approved_)  
3. The rule executes **asynchronously** to avoid performance issues.  
4. It looks up the employee’s **Leave Bucket** record using:
   - `employee (requested_by)`
   - `leave_type`

If a matching record is found, it:

- Deducts the requested leave days (`duration`) from **balance**  
- Adds the same days to **taken**  
- Updates the **Leave Bucket** table  

---

#  Purpose

- Keeps the **Leave Bucket** table in sync with actual requests  
- Automates leave deduction when requests are approved  
- Reduces **manual errors** in leave tracking  

---

##  Script Logic
```javascript
(function executeRule(current, previous /*null when async*/) {

    // Query the Leave Bucket table for the employee and leave type
    var gr = new GlideRecord('x_872769_flexile_0_leave_bucket_5');
    gr.addQuery('employee', current.requested_by);
    gr.addQuery('leave_type', current.leave_type);
    gr.query();

    // If a matching leave bucket exists, update the balances
    if (gr.next()) {
        gr.balance = gr.balance - current.duration;  // Deduct from available balance
        gr.taken = gr.taken + current.duration;      // Increase taken leave count
        gr.update();                                 // Save changes
    }

})(current, previous);
