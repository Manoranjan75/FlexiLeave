##  Business Rule: Prevent Duplicate Leave Requests (Before)

This Business Rule in the **FlexiLeave** ServiceNow application ensures data consistency by preventing duplicate leave requests for the same employee within a given date range. It uses asynchronous execution so the validation logic runs after a record insert without slowing down the main transaction.

---

###  Business Rule Details

- **Table:** `x_872769_flexile_0_leave_request_5` (Leave Request table)  
- **Application:** FlexiLeave  
- **Execution:** Runs **Async** (after record insertion)  
- **Trigger:** `onInsert` event  
- **Condition:** Executes only if **Start Date** and **End Date** are not empty  

---

###  How It Works

When a new leave request is inserted, the rule performs the following checks:

1. Checks if the **same employee** (`requested_by`) already has an existing leave request.  
2. Checks if the **leave dates** (`start_date` and `end_date`) overlap with an existing request.  

If a duplicate leave request is found:

- An **error message** is displayed to the user.  
- The **insert action is aborted** using `current.setAbortAction(true)`, preventing duplicate records.  

---

###  Key ServiceNow Components Used

- **GlideRecord** – Queries the Leave Request table to find duplicate records.  
- **gs.addErrorMessage()** – Displays a warning message to the user.  
- **current.setAbortAction(true)** – Stops further execution and blocks the creation of the record.

---

###  Code Explanation

```javascript
(function executeRule(current, previous /*null when async*/) { 
    // Create a GlideRecord object for the Leave Request table
    var gr = new GlideRecord('x_872769_flexile_0_leave_request_5'); 
    
    // Check if another request exists for same employee and date range
    gr.addQuery('requested_by', current.requested_by); 
    gr.addQuery('start_date', current.start_date); 
    gr.addQuery('end_date', current.end_date); 
    gr.addQuery('sys_id', '!=', current.sys_id); // Exclude the current record
    
    gr.query(); 
    
    // If duplicate exists, stop the action and show error
    if (gr.next()) { 
        gs.addErrorMessage('A leave request already exists for this date range.'); 
        current.setAbortAction(true); 
    } 
})(current, previous);
