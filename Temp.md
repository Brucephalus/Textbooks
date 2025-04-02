Here’s a **Power Automate flow example** for handling JSON files in a **SharePoint Document Library**. This flow will:

1. **Trigger when a new JSON file is uploaded** to the library.
2. **Read the file contents**.
3. **Parse the JSON** to extract key metadata fields.
4. **Update the file's metadata columns** in the document library.
5. **Optionally send a notification** if a certain condition (e.g., "High Priority") is met.

---

## **Step-by-Step Power Automate Flow: Process JSON in a SharePoint Library**
### **Flow Trigger:**  
- **Trigger: "When a file is created (properties only)"**
  - Site Address: *(Your SharePoint site)*
  - Library Name: *(Your document library name)*

---

### **Step 1: Get the JSON File Content**
- **Action: "Get file content"**
  - Site Address: *(Same as above)*
  - File Identifier: `Identifier` *(From trigger output)*

---

### **Step 2: Parse JSON**
- **Action: "Parse JSON"**
  - Content: `File Content` *(From "Get file content")*
  - Schema: *(Click "Generate from sample" and paste a sample JSON structure)*

#### **Example Schema:**
```json
{
    "type": "object",
    "properties": {
        "schemaVersion": { "type": "string" },
        "recordType": { "type": "string" },
        "metadata": {
            "type": "object",
            "properties": {
                "recordId": { "type": "string" },
                "priority": { "type": "string" },
                "createdOn": { "type": "string", "format": "date-time" },
                "tags": { "type": "array", "items": { "type": "string" } }
            }
        },
        "payload": {
            "type": "object",
            "properties": {
                "customerName": { "type": "string" },
                "description": { "type": "string" }
            }
        }
    }
}
```
---

### **Step 3: Extract Key Metadata Fields**
- **Action: "Initialize Variable"**
  - Name: `RecordType`
  - Type: `String`
  - Value: `@{body('Parse_JSON')?['recordType']}`

- **Action: "Initialize Variable"**
  - Name: `Priority`
  - Type: `String`
  - Value: `@{body('Parse_JSON')?['metadata']?['priority']}`

- **Action: "Initialize Variable"**
  - Name: `RecordID`
  - Type: `String`
  - Value: `@{body('Parse_JSON')?['metadata']?['recordId']}`

---

### **Step 4: Update File Properties in SharePoint**
- **Action: "Update file properties"**
  - Site Address: *(Your SharePoint site)*
  - Library Name: *(Your document library)*
  - ID: `ID` *(From trigger output)*
  - **Record Type**: `RecordType`
  - **Priority**: `Priority`
  - **Record ID**: `RecordID`
  - **Tags**: `@{join(body('Parse_JSON')?['metadata']?['tags'], ', ')}` *(Convert array to comma-separated string)*

---

### **Step 5: Conditional Action (Optional: Send Notification for High Priority)**
- **Condition: "Priority is High"**
  - Expression: `@equals(variables('Priority'), 'High')`

✅ **If Yes** (Priority = High):
- **Action: "Send Teams Message / Email Notification"**
  - Message: `🚨 High-Priority Issue Created: @{variables('RecordID')}`
  - Recipients: *(Your support team or relevant users)*

🚫 **If No** (Priority is Low/Medium): Do nothing.

---

### **Step 6: Log the Process (Optional)**
- **Action: "Append to String Variable"**
  - Name: `ProcessingLog`
  - Value: `Processed JSON File: @{variables('RecordID')}, Priority: @{variables('Priority')}`

- **Action: "Create Item in SharePoint List (Optional, for Audit Logs)"**
  - Site Address: *(Your site)*
  - List Name: `"Processing Logs"`
  - Title: `Processed @{variables('RecordID')}`
  - Status: `"Completed"`
  - Timestamp: `utcNow()`

---

## **Flow Summary**
✅ Triggers when a JSON file is uploaded to SharePoint.  
✅ Reads and parses the JSON data.  
✅ Extracts and updates metadata fields in SharePoint.  
✅ Sends alerts if priority = High.  
✅ Logs the processing results (optional).

---

### **Why This Flow Works Well**
✔ **Scalability** – Works even if JSON formats vary.  
✔ **Automation** – Ensures metadata is filled correctly without manual input.  
✔ **Performance** – Efficiently queries JSON fields without overloading SharePoint.  
✔ **Extensibility** – Can easily add additional logic (e.g., archive older files).  

---

Would you like a **Power Automate template file** for quick import, or do you need any modifications for your specific use case? 🚀
