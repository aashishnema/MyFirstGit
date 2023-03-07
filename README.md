# Update a date filter to a specific range

API: Extensions API\
Difficulty: ⚫⚫⚪⚪\
Requirements: Text editor, Web hosting or local web server

This mini project will show you how to set up an extension that will automatically update a date range filter to a lower bound of one year ago, and an upper bound of right now. Extensions are a way to embed web applications into Tableau dashboards that allow you to extend the features or Tableau and integrate with other applications.

### Set up your environment
In this mini project, we will be utilizing Glitch as our text editor and hosting service.
1. Create your own copy of the starter project by [remixing it](https://glitch.com/edit/#!/remix/datadev-date-range-filter). 
2. To keep your project and make all your projects easy to find later click **Sign In** to create an account.
3. Click on the **Show live** link at the top to go to your project page.
4. Copy the URL on that page, it should look something like: `https://substantial-girl.glitch.me/`.
5. Close the live preview page and go back to the editor. Click on MyAwesomeExtension.trex to open it. Change `YOUR URL HERE` to the URL you copied in the last step.
6. Download the MyAwesomeExtension.trex file by editing and pasting the below link into your browser window.
`<YOUR URL HERE>/MyAwesomeExtension.trex`. The file should download into your downloads folder.

### Start Desktop in debug mode
###### Windows
1. Exit Tableau if it is already running on your computer.
2. Open a Command Prompt window.
3. Start Tableau using the following command. Replace <version> with the version of Tableau you are using (for example, Tableau 2019.1).
```batch
"C:\Program Files\Tableau\Tableau <version>\bin\tableau.exe" --remote-debugging-port=8696
```

###### Mac
1. Open a Terminal window.
2. Start Tableau using the following command. Replace <version> with the version of Tableau you are using (for example,2019.1.app).
```batch
open /Applications/Tableau\ Desktop\ <version>.app --args --remote-debugging-port=8696
```

### Test your extension
1. Download and open [DateRange.twbx](https://tableau.github.io/datadev-hackathon/Extensions/DateRange/DateRange.twbx) in the Desktop instance you just opened with the debugger. (File > Open, not just double click)
2. Drag and drop a new **Extension** object (bottom left pane) to the bottom of your dashboard.
3. Allow the extension to run.
4. Confirm that you see "This is an extension!".
5. Go back to Glitch and change "This is an extension!" to a different phrase.
6. Reload your extension and make sure you see your phrase in Tableau.

### Start coding!
1. In Glitch, go to your index.html file and before the `</head>` line, add a line to reference the Tableau Extensions API library:
```html
<script src="https://cdn.jsdelivr.net/gh/tableau/extensions-api/lib/tableau-extensions-1.latest.js"></script>
```
2. Next, go to the script.js file (which is already referenced in index.html) and add the following to initialize your extension:
```javascript
tableau.extensions.initializeAsynch().then(() => {
  console.log('I have been initialized!!')
});
```

### Wait! Let's use the debugger
1. In your browser, go to the [debugging homepage](http://localhost:8696) (`http://localhost:8696`).\
_If you are using Desktop 2018.3 or lower please follow the instructions [here](https://tableau.github.io/extensions-api/docs/trex_debugging.html#download-the-chromium-browser) to download the Chromium browser for debugging._
2. Click on your extension in the browser (should be named "My Awesome Extension").
3. If it's not already selected, click the **Console** tab at the top to open the console.
4. Notice we're getting an error saying `Uncaught TypeError: tableau.extensions.initializeAsynch is not a function at script.js:1`. Looks like there is a typo in our call to initialize. Remove the 'h' from the end of the function call initializeAsynch -> initializeAsync in script.js.
```javascript
tableau.extensions.initializeAsync().then(() => {
  console.log('I have been initialized!!')
});
```
5. Reload the extension in Tableau by clicking the down arrow on the extension zone and selecting **Reload** from the menu.
6. In your browser with the debugger, go to `http://localhost:8696` again and click on **My Awesome Extension**. (Note you must reload the page, hitting back alone will not reset the console)
7. You should now see the message "I have been initialized!!" in the console. Use this console to test and debug your extension as you work.

### Automatically set a date range filter.
Our goal in this section will be to set the date range filter to a lower bound of one year ago, and an upper bound of today.
1. Make sure you have the latest data to work with by refreshing the extract. **Data** > **Historical AMZN Data** > **Extract** > **Refresh**\
_Note: You will not see the latest data in the viz until you adjust the filter._
2. Before we begin, familiarize yourself with the [Extensions API Reference](https://tableau.github.io/extensions-api/docs/index.html). All filter-setting functions stem from worksheet objects. We will be using [applyRangeFilterAsync()](https://tableau.github.io/extensions-api/docs/interfaces/worksheet.html#applyrangefilterasync) in this example to change the filter.
3. Notice that applyRangeFilterAsync() takes two parameters:
    * fieldName: this takes a string of the field you want to filter.
    * filterOptions: this takes a [RangeFilterOptions](https://tableau.github.io/extensions-api/docs/interfaces/rangefilteroptions.html) object. This object takes the optional min, max, and nullOption parameters.
4. Since applyRangeFilterAsync() is a method on a worksheet we first have to get a worksheet to call it on. But to get the worksheet, first, we have to get the dashboard. After initializing we can get access to the dashboard by adding the following line into the .then() block replacing the console log: [`let dashboard = tableau.extensions.dashboardContent.dashboard;`](https://tableau.github.io/extensions-api/docs/index.html)
5. Then from there, we can get the desired worksheet by adding the following line next: `let selectedWorksheet = dashboard.worksheets.find(w => w.name === 'Historical Trend');`. Here we're using the [**find()**](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/find) function to choose a specific worksheet. _Note: Try console logging **dashboard** and **worksheet** to see their structure._
6. At this point your script.js file should look something like this:
```javascript
tableau.extensions.initializeAsync().then(() => {
    let dashboard = tableau.extensions.dashboardContent.dashboard;
    let selectedWorksheet = dashboard.worksheets.find(w => w.name === 'Historical Trend');
});
```
7. Let's add a function to do the work of getting the right dates and setting the filter. Below the entire initialize function add a new [function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions): 
```javascript
function updateFilterRange(worksheet, fieldName) {

}
```
_Note: our new function takes two parameters of it's own, worksheet and fieldName._
8. Inside this method we will get [today's date](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date) & [subtract a year](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/setFullYear) from it for our minimum.
```javascript
let today = new Date();
let lastYear = new Date();
lastYear.setFullYear(today.getFullYear()-1);
```
9. Now that we have the two dates we need for our min and max parameters (nullOption is optional) we can use applyRangeFilterAsync(): `worksheet.applyRangeFilterAsync(fieldName, { min: lastYear, max: today});`
10. At this point your script.js file should look something like this:
```javascript
tableau.extensions.initializeAsync().then(() => {
    let dashboard = tableau.extensions.dashboardContent.dashboard;
    let selectedWorksheet = dashboard.worksheets.find(w => w.name === 'Historical Trend');
});

function updateFilterRange(worksheet, fieldName) {
    let today = new Date();
    let lastYear = new Date();
    lastYear.setFullYear(today.getFullYear()-1);
    worksheet.applyRangeFilterAsync(fieldName, { min: lastYear, max: today});
}
```
11. Now that we have our function, let's call it when we initialize. Add the following into your initialize function: 
```javascript
let fieldName = 'Date';
updateFilterRange(selectedWorksheet, fieldName);
```
12. Finally, your script.js file should look like this:
```javascript
tableau.extensions.initializeAsync().then(() => {
    let dashboard = tableau.extensions.dashboardContent.dashboard;
    let selectedWorksheet = dashboard.worksheets.find(w => w.name === 'Historical Trend');
    let fieldName = 'Date';
    updateFilterRange(selectedWorksheet, fieldName);
});

function updateFilterRange(worksheet, fieldName) {
    let today = new Date();
    let lastYear = new Date();
    lastYear.setFullYear(today.getFullYear()-1);
    worksheet.applyRangeFilterAsync(fieldName, { min: lastYear, max: today});
}
```
13. Test your extension by setting the date range filter to month range and then reload your extension. Notice the range filter changes to a lower bound of one year ago, and an upper bound of today and the viz updates.

### Challenge exercises
Now that you know the basics of updating date range filters, try implementing the following bonus challenges.
1. Add a filter for the closing price and automatically update it to a certain range.
2. Set the filter to one week ago or one month ago.
3. Add buttons that set the date to specific ranges, like YTD, last week etc.
4. Use two parameters for the range.