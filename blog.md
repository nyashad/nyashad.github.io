---
layout: post
title: "A script that integrates Elementor Forms with Google Sheets via Webhooks"
date: 2024-09-25 
categories: blog
post-image: images/Integrating-forms-to-google-sheets-via-webhook.jpg
---

# Streamlining Form Submissions from Elementor to Google Sheets

If you're using Elementor for your website and need an efficient way to store form submissions, integrating Google Sheets via a webhook is a powerful solution. This setup ensures that all form data is automatically recorded in a structured way, making it easy to analyze and manage.

## Why Use Google Sheets?
- **Real-time data storage**: Automatically syncs new form entries.
- **Easy data manipulation**: Sort, filter, and analyze responses without hassle.
- **No coding required**: Simple setup using a webhook.

## Setting Up the Integration
Follow these steps to connect Elementor forms with Google Sheets using a webhook:

### Step 1: Create a Google Sheet
1. Open Google Sheets.
2. Create a new sheet and label the columns to match your Elementor form fields.

### Step 2: Set Up a Google Apps Script
1. Click **Extensions > Apps Script** in Google Sheets.
2. Replace the existing script with the following:

```javascript
/*
 * To activate this script, follow these steps:
 
   ***   Open the Google Sheet you wish to use,
    **   Go to the "Tools" menu and select "Script Editor"
     *   Paste this script into the editor and click "Save".
     *   
   ***   Next, click: 
    **     "Publish" and choose "Deploy as web app..."
   ***   For the settings, select: 
    **      Execute the app as: Me (youremail@gmail.com)
    **      Who has access: Anyone, including anonymous users
   *                           - Depending on your Google Apps configuration, this option might not be available. If so, contact your Google Apps admin, or use a personal Gmail account.
   *
   ***    Click "Deploy". You may need to grant permissions at this point.
   *                           - You might see a warning; click "Advanced" in the bottom left and proceed.
   *      
   ***    The URL generated will serve as the webhook for your Elementor form.
   *                            - To test, paste the URL into your browser. You should see the message: "Yepp this is the webhook URL, request received."
   *         
   ***    EMAIL NOTIFICATIONS: 
            *  By default, email notifications are disabled.
            *  To enable them:
              *  On line 37, change "false" to "true"
              *  On line 40, replace "Change_to_your_Email" with your email address (keeping the quotes)
              *  Save the script again. Now you're set to collect those leads!
 */
function doGet(e) {}
function doPost(e) {
  var o = e.parameter;
  SpreadsheetApp.getActiveSheet().appendRow([ o.name, o.email, o.message, e ]);
}

// Change to true to enable email notifications
var emailNotification = false;

// Enter your email address below (keep the quotation marks!) 
var emailAddress = "youremail@gmail.com";

/**
 * Google app-script to utilize Elementor Pro Forms webhook
 * For Usage see: https://github.com/pojome/elementor/issues/5894
 * Script updated and maintained by Nyashadzashe Ndhlovu
 */
 
//////////////////////////////////////////////////////////////////////////////////////////////////////////////////
// DO NOT EDIT THESE NEXT PARAMS
var isNewSheet = false;
var receivedData = [];

/**
 * This function fires when the web app receives a GET request.
 * Not used but required.
 */
function doGet( e ) {
  return HtmlService.createHtmlOutput( "Yepp this is the webhook URL, request received" );
}

// Webhook Receiver - triggered with form webhook to published App URL.
function doPost( e ) {
  var params = JSON.stringify(e.parameter);
  params = JSON.parse(params);
  insertToSheet(params);

  // HTTP Response
  return HtmlService.createHtmlOutput( "post request received" );
}

// Flattens a nested object for easier use with a spreadsheet
function flattenObject( ob ) {
  var toReturn = {};
  for ( var i in ob ) {
    if ( ! ob.hasOwnProperty( i ) ) continue;
    if ( ( typeof ob[ i ] ) == 'object' ) {
      var flatObject = flattenObject( ob[ i ] );
      for ( var x in flatObject ) {
        if ( ! flatObject.hasOwnProperty( x ) ) continue;
        toReturn[ i + '.' + x ] = flatObject[ x ];
      }
    } else {
      toReturn[ i ] = ob[ i ];
    }
  }
  return toReturn;
}

// Normalize headers
function getHeaders( formSheet, keys ) {
  var headers = [];
  
  // Retrieve existing headers
  if ( ! isNewSheet ) {
    headers = formSheet.getRange( 1, 1, 1, formSheet.getLastColumn() ).getValues()[0];
  }

  // Add any additional headers
  var newHeaders = [];
  newHeaders = keys.filter( function( k ) {
    return headers.indexOf( k ) > -1 ? false : k;
  } );

  newHeaders.forEach( function( h ) {
    headers.push( h );
  } );
  return headers;
}

// Normalize values
function getValues( headers, flat ) {
  var values = [];
  // Push values based on headers
  headers.forEach( function( h ){
    values.push( flat[ h ] );
  });
  return values;
}

// Insert headers
function setHeaders( sheet, values ) {
  var headerRow = sheet.getRange( 1, 1, 1, values.length );
  headerRow.setValues( [ values ] );
  headerRow.setFontWeight( "bold" ).setHorizontalAlignment( "center" );
}

// Insert Data into Sheet
function setValues( sheet, values ) {
  var lastRow = Math.max( sheet.getLastRow(), 1 );
  sheet.insertRowAfter( lastRow );
  sheet.getRange( lastRow + 1, 1, 1, values.length ).setValues( [ values ] ).setFontWeight( "normal" ).setHorizontalAlignment( "center" );
}

// Find or create sheet for form
function getFormSheet( formName ) {
  var formSheet;
  var activeSheet = SpreadsheetApp.getActiveSpreadsheet();

  // Create sheet if needed
  if ( activeSheet.getSheetByName( formName ) == null ) {
    formSheet = activeSheet.insertSheet();
    formSheet.setName( formName );
    isNewSheet = true;
  }
  return activeSheet.getSheetByName( formName );
}

// Main function where it all happens
function insertToSheet( data ) {
  var flat = flattenObject( data );
  var keys = Object.keys( flat );
  var formName = data["form_name"];
  var formSheet = getFormSheet( formName );
  var headers = getHeaders( formSheet, keys );
  var values = getValues( headers, flat );
  setHeaders( formSheet, headers );
  setValues( formSheet, values );
	
  if ( emailNotification ) {
    sendNotification( data, getSheetURL() );
  }
}

function getSheetURL() {
  var spreadsheet = SpreadsheetApp.getActiveSpreadsheet();
  var sheet = spreadsheet.getActiveSheet();
  return spreadsheet.getUrl();
}

function sendNotification( data, url ) {
  var subject = "A new Elementor Pro Forms submission has been inserted into your sheet";
  var message = "A new submission has been received via " + data['form_name'] + " form and inserted into your Google sheet at: " + url;
  MailApp.sendEmail( emailAddress, subject, message, {
    name: 'Automatic Emailer Script'
  } );
}
```

3. Click the **Deploy** button and select **New Deployment**.
4. Under **Select type**, choose **Web app**.
5. Set **Who has access** to **Anyone**.
6. Copy the generated URL; this will be your webhook endpoint.

### Step 3: Configure Elementor Webhook
1. In Elementor, edit your form.
2. Go to the **Actions After Submit** section.
3. Click **Add Action** and select **Webhook**.
4. Paste the webhook URL from Google Apps Script.
5. Map the form fields to match your Google Sheets column headers.
6. Save the form and test by submitting an entry.

## Verifying the Integration
- Check your Google Sheet to confirm that new form submissions are automatically recorded.
- If you encounter issues, review the **Apps Script execution logs** for errors.

## Conclusion
Integrating Elementor forms with Google Sheets via a webhook is a seamless way to store and manage data. This approach eliminates manual entry and ensures real-time updates. You can further enhance the integration by adding email notifications or data validation scripts.

Check out the full project on GitHub: [nyashad/elementor-googlesheets-webhook](https://github.com/nyashad/elementor-googlesheets-webhook).
