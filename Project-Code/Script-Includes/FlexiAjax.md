###  Script Include: FlexiAjax (Client-Callable)

The `FlexiAjax` Script Include allows client scripts to fetch:

1. A **numeric date difference** between two provided dates.  
2. An employee’s **leave bucket details** (`accrued`, `balance`, `taken`) for a chosen leave type.

It uses `AbstractAjaxProcessor` so client scripts can call server-side logic asynchronously via `GlideAjax`, passing parameters like:

- `sysparm_start`
- `sysparm_end`
- `sysparm_user`
- `sysparm_leavetype`

The server returns a **lightweight string response** that can be used to populate form fields or perform validations on the fly.

---

###  Notes for Use

- Mark the Script Include as **client callable**.
- Call it from a **Client Script** using `GlideAjax` with `sysparm_name` set to `getDatediff` or `getLeaveBucket`.
- `getDatediff` returns **milliseconds**; convert to days on the client as needed.
- `getLeaveBucket` returns a **JSON string**; parse it on the client to populate `accrued`, `balance`, and `taken` fields.


---

###  Code

```javascript
var FlexiAjax = Class.create();
FlexiAjax.prototype = Object.extendsObject(global.AbstractAjaxProcessor, {

    // Returns the difference between two dates as a numeric value (milliseconds)
    getDatediff: function() {
        var d1 = new GlideDate();
        d1.setDisplayValue(this.getParameter('sysparm_start'));
        var d2 = new GlideDate();
        d2.setDisplayValue(this.getParameter('sysparm_end'));

        var duration = GlideDate.subtract(d1, d2);
        return duration.getNumericValue(); // milliseconds
    },

    // Returns JSON with leave bucket info for an employee and leave type
    getLeaveBucket: function() {
        var gr = new GlideRecord('x_872769_flexile_0_leave_bucket_5');
        gr.addQuery('employee', this.getParameter('sysparm_user'));
        gr.addQuery('leave_type', this.getParameter('sysparm_leavetype'));
        gr.query();
        if (gr.next()) {
            var leaveDetails = {};
            leaveDetails.accrued = '' + gr.accrued;
            leaveDetails.balance = '' + gr.balance;
            leaveDetails.taken   = '' + gr.taken;
            return JSON.stringify(leaveDetails);
        }
    },

    type: 'FlexiAjax'
});
