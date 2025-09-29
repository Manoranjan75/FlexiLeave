#  Business Rule: Create Leave Bucket (Async)

The **Create Leave Bucket** business rule is responsible for initializing leave balances for employees when a new record is inserted.  
It automatically creates a Leave Bucket entry based on the employee’s country-specific leave policy defined in the **Leave Calculator** table.

---

## Configuration Details

- **Table**: Create Leave Bucket  
- **Application**: FlexiLeave  
- **Execution**: Runs Async after record insertion  
- **Trigger**: On Insert event  

---


##  How It Works

When a new employee record is inserted, the rule looks up **Leave Calculator** using the employee’s country.

Based on the configuration (`leave_assignment`):

- **Yearly (`y`)** → Calculates total leaves for the remaining months of the year  
- **Monthly (`m`)** → Assigns leave balance on a monthly basis  

It then creates a new **Leave Bucket** entry with:

- `leave_type`  
- `accrued` leave  
- `balance` (initially equal to accrued)  
- `taken = 0`  
- `employee = current.sys_id`  

---

##  Purpose

- Automates the initial creation of leave balances for employees  
- Supports both **yearly** and **monthly** leave allocation models  
- Ensures employees’ leave data is initialized correctly **without manual intervention**

---


##  Script Logic

```javascript
(function executeRule(current, previous /*null when async*/) {

    // Query the Leave Calculator table based on employee's country
    var gr = new GlideRecord('x_872769_flexile_0_leave_calculator_5');
    gr.addQuery('country.name', current.location.country);
    gr.query();

    if (gr.next()) {
        // If yearly assignment
        if (gr.leave_assignment == 'y') {
            var gdt = new GlideDateTime();
            var currentMonth = gdt.getDayOfMonthLocalTime();
            var monthsLeft = 12 - currentMonth;
            var perMonth = gr.leaves / 12;
            var totalLeaves = perMonth * monthsLeft;

            var leaveBucket = new GlideRecord('x_872769_flexile_0_leave_bucket_5');
            leaveBucket.initialize();
            leaveBucket.leave_type = gr.leave_type;
            leaveBucket.accrued = totalLeaves;
            leaveBucket.balance = totalLeaves;
            leaveBucket.taken = 0;
            leaveBucket.employee = current.sys_id;
            leaveBucket.insert();
        }

        // If monthly assignment
        else if (gr.leave_assignment == 'm') {
            var perMonthM = gr.leaves / 12;

            var leaveBucketM = new GlideRecord('x_872769_flexile_0_leave_bucket_5');
            leaveBucketM.initialize();
            leaveBucketM.leave_type = gr.leave_type;
            leaveBucketM.accrued = perMonthM;
            leaveBucketM.balance = perMonthM;
            leaveBucketM.taken = 0;
            leaveBucketM.employee = current.sys_id;
            leaveBucketM.insert();
        }
    }

})(current, previous);
