###  OnChange Client Script: Set Duration for Half Day

This **onChange Client Script** sets the **duration** field to **0.5** whenever the **Half Day** checkbox is checked.  
It uses a lightweight client-side guard with `isLoading` and reads the checkbox as a string value, which is standard behavior for `g_form.getValue` on boolean fields in ServiceNow.

---

####  What It Does

- Fires when the **Half Day** checkbox changes.
- Exits during form load or when the value is empty to avoid unnecessary execution (standard onChange guard pattern).
- Reads the checkbox value (returns `'true'` or `'false'` as strings via `g_form.getValue`).
- Sets the **duration** field to `0.5` if the checkbox is checked.
- Executes immediately on the client without server calls for a responsive experience.

---

####  Client Script Code

```javascript
function onChange(control, oldValue, newValue, isLoading, isTemplate) {
    if (isLoading || newValue === '') {
        return;
    }

    // When Half Day is checked, set duration to 0.5
    var half = g_form.getValue('half_day'); // returns 'true' or 'false'
    if (half === 'true') {
        g_form.setValue('duration', '0.5');
    }
}
