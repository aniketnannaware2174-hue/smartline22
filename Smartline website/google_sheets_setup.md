# Google Sheets Enquiry Setup Guide

To save your website form enquiries directly into a Google Sheet securely, follow these simple steps to deploy the Google Apps Script. This will generate a secret "Web App URL" that we will use to connect your website to your sheet.

## Step 1: Create the Google Sheet
1. Open [Google Sheets](https://sheets.google.com) and create a new Blank spreadsheet.
2. Name the sheet something like "Smartline Form Enquiries".
3. In the first row, add the EXACT following headers in columns A through F:
   * **A1:** `timestamp`
   * **B1:** `name`
   * **C1:** `email`
   * **D1:** `phone`
   * **E1:** `message`
   * **F1:** `attachment`

*(Note: File attachments will be sent as base64 text or we can ignore them for the sheet. The basic text fields are the most critical).*

## Step 2: Add the Apps Script
1. In your new Google Sheet, click on **Extensions** in the top menu, then select **Apps Script**.
2. A new tab will open with a code editor. **Delete all the code** that is in there.
3. **Copy and paste** the following code into the editor:

```javascript
var sheetName = 'Sheet1';
var scriptProp = PropertiesService.getScriptProperties();

function intialSetup () {
  var activeSpreadsheet = SpreadsheetApp.getActiveSpreadsheet();
  scriptProp.setProperty('key', activeSpreadsheet.getId());
}

function doPost (e) {
  var lock = LockService.getScriptLock();
  lock.tryLock(10000);

  try {
    var doc = SpreadsheetApp.openById(scriptProp.getProperty('key'));
    var sheet = doc.getSheetByName(sheetName);

    var headers = sheet.getRange(1, 1, 1, sheet.getLastColumn()).getValues()[0];
    var nextRow = sheet.getLastRow() + 1;

    var newRow = headers.map(function(header) {
      return header === 'timestamp' ? new Date() : e.parameter[header] || '';
    });

    sheet.getRange(nextRow, 1, 1, newRow.length).setValues([newRow]);

    return ContentService
      .createTextOutput(JSON.stringify({ 'result': 'success', 'row': nextRow }))
      .setMimeType(ContentService.MimeType.JSON);
  }

  catch (e) {
    return ContentService
      .createTextOutput(JSON.stringify({ 'result': 'error', 'error': e }))
      .setMimeType(ContentService.MimeType.JSON);
  }

  finally {
    lock.releaseLock();
  }
}
```

4. Click the **Save** icon (the floppy disk) or press Ctrl+S.
5. In the toolbar above the code, click the "Run" button while the `intialSetup` function is selected in the dropdown. 
6. It will ask for permissions ("Review Permissions"). Click it, choose your Google account, click "Advanced", and then "Go to Untitled project (unsafe)". Click "Allow". This authorizes the script to edit this specific Google Sheet.

## Step 3: Deploy to get your Web App URL
1. In the top right corner of the Apps Script editor, click the big blue **Deploy** button and select **New deployment**.
2. Click the gear icon ⚙️ next to "Select type" and choose **Web app**.
3. Fill out the form exactly like this:
   * **Description:** Form integration
   * **Execute as:** `Me (your email)`
   * **Who has access:** `Anyone` *(This is required so the public website can send data to it).*
4. Click **Deploy**.
5. Once it finishes, it will give you a **Web app URL** that looks like this: `https://script.google.com/macros/s/.../exec`.

## Final Step
**Copy that Web App URL** and paste it back in our chat here! 

Once I have that URL, I will add it to [index.html](file:///c:/Users/pravi/Desktop/Smartline%20website/smartline%20web/index.html), `contact.html`, and any other pages so that clicking "Submit" on your website immediately sends the data to your private Google Sheet.
