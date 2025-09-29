###  OnChange Client Script: Populate Leave Bucket Fields

This **onChange Client Script** populates `accrued`, `balance`, and `taken` when the **Leave Type** field changes.  
It calls the client-callable Script Include **FlexiAjax** via `GlideAjax`, parses the JSON response, and updates form fields asynchronously without blocking the UI.

---

####  What It Does

- Triggers on `leave_type` change.
- Skips execution during **form load** or if the value is empty (common guard pattern in ServiceNow client scripts).
- Uses `GlideAjax` with `sysparm_name` and additional `sysparm_*` parameters to invoke the **getLeaveBucket** function in the Script Include.
- Handles the callback to update form fields asynchronously.
- Converts the XML answer into JSON using `JSON.parse`.
- Updates multiple fields with `g_form.setValue`.

---

#### Why It’s Correct

- Asynchronous `getXML` keeps the UI **responsive** while the server processes the request.
- Passing `sysparm_name` is **mandatory** to select the Script Include function.
- Additional parameters carry user and leave type context to the server, following **GlideAjax best practices**.
- Parsing JSON allows setting several related values in **one server call**.

---

####  Client Script Code

```javascript
function onChange(control, oldValue, newValue, isLoading, isTemplate) {
    if (isLoading || newValue === '') {
        return;
    }

    // Call server to get leave bucket for selected leave type
    var ga = new GlideAjax('FlexiAjax');
    ga.addParam('sysparm_name', 'getLeaveBucket');
    ga.addParam('sysparm_user', g_user.userID);
    ga.addParam('sysparm_leavetype', newValue);
    ga.getXML(getBucket);

    function getBucket(response) {
        var answer = response.responseXML.documentElement.getAttribute("answer");
        var result = JSON.parse(answer);
        g_form.setValue('accrued', result.accrued);
        g_form.setValue('balance', result.balance);
        g_form.setValue('taken', result.taken);
    }
}
