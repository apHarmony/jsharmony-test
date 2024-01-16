# ==============
# jsharmony-test
# ==============

Testing tools for jsHarmony

## Installation

1. Ensure that the jsharmony-cli is installed with
```
npm install -g jsharmony-cli
```

2. Install jsharmony-test in the project folder
```
npm install jsharmony-test --save
```

## Setup
1. Create a folder in the project root named "test"
2. Create a subfolder for the test scripts, ex. "test/module1"
2. Create a _config.json file in the "test/module1" folder with the test configuration:
```javascript
{
  "server": "https://server:port", // Application URL, ex: https://localhost:8000
  "appbasepath": "<path-to-test>", //(Optional) Path to application base, defaults to current working directory
  "datadir": "---", //(Optional) Path to application data folder, defaults to appbasepath/data
  "screenshot": { ... } //(Optional) Screenshot configuration, if screenshots are taken.  Full screenshot configuration options are listed under the screenshot command below.
  "namespace": "", //(Optional) Root namespace
  "require": [] //(Optional) Run the tests in this array before any other tests in this test set, used in scenarios such as login / authentication
  "before": [ COMMAND1, COMMAND2, ... ] //(Optional) Array of commands to run before running any other tests in this test set (command syntax defined below)
  "after": [ COMMAND1, COMMAND2, ... ] //(Optional) Array of commands to run after running the tests in this test set (command syntax defined below)
}
```
3. Create a .json file for each test script within the test folder, ex "test/module1/test1.json"
```javascript
{
  "id": "TEST_ID",
  "title": "Test Name",
  "batch": "01_Test",  //(Optional) Batch ID used to sort test sequence
  "require": [ "TEST_ID1", "TEST_ID2" ],   //(Optional) Array of tests that must be executed before this test. Used to sort tests
  "commands": [ COMMAND1, COMMAND2, ... ],  //Array of commands to run within this test script (command syntax defined below)
}
```
4. The "test" folder should now appear as follows
```bash
└───test
    └───module1
            _config.json
            test1.json
```

## Running Tests

1. Create the test master screenshots
```
jsharmony test master screenshots test\handheld
```
* Optionally use the --show-browser argument to preview in the browser
2. Run the tests, to compare the current application against the master screenshots
```
jsharmony test screenshots test\handheld
```
* Optionally use the --show-browser argument to preview in the browser

## Commands
#### Overview
Commands are defined as JSON.  Each command requires an "exec" parameter to define the command name, and can optionally have a "timeout" parameter to specify a maximum timeout for that command.  The COMMAND_NAME may be any of the commands described below.
```javascript
{ "exec": "COMMAND_NAME", "timeout": 10000 }
```
#### Command: Navigate
The "navigate" command is used to redirect the test browser to a new URL.
```javascript
{ "exec": "navigate", "url": "https://localhost:3000" }
```
* The url property can use variables, e.g., "https://localhost:3000/App?token=@TOKEN"

#### Command: Screenshot
The "screenshot" command is used to take a screenshot of the current browser window.
```javascript
{ "exec": "screenshot", "id": "login_start" }

//The following optional screenshot parameters are available:
{
  "x": 0,         //Take screenshot starting at X position
  "y": 0,         //Take screenshot starting at Y position
  "width": 950,   //Set browser width before taking screenshot
  "height": 700,  //Set browser height before taking screenshot
  "beforeScreenshot": "",  // Execute server-side JS code before taking screenshot
  "onload": "",   // Execute client-side JS code before taking screenshot
  "cropToSelector": ".container", // Crop screenshot to DOM selector
  "postClip":   { // Clip screenshot to target size
    "x": 1,
    "y": 1,
    "width": 1,
    "height": 1
  },
  "trim": true,   // Trim blank pixels from edge of screenshot
  "exclude": [    // Draw a black box over target areas, to prevent false positive errors in screenshot comparison
    {  //Exclude region based on absolute rectangle
      "x": 1,
      "y": 1,
      "width": 1,
      "height": 1
    },
    {  //Exclude region based on DOM selector
      "selector": ""
    }
  ]
}
```

#### Command: Wait
The "wait" command pauses test execution until a specified condition is met.
```javascript
{ "exec": "wait" }

//Example usage:
{ "exec": "wait", "element": ".homePage" }
{ "exec": "wait", "text": "Welcome!" }
```
* Optional Parameters:
  * "element": [ELEMENT_SELECTOR](#ELEMENT_SELECTOR)   // Selector for the target element
  * "text": [TEXT_COMPARE](#TEXT_COMPARE)         // The element or any child element containing the text (defined below)
    * If no element is specified, wait for any child of the document containing the text
  * "while_waiting": [ COMMAND1, COMMAND2 ]  // Execute commands while the wait operation is in progress

#### Command: Input
The "input" command performs key presses or enters data into elements.
```javascript
{ "exec": "input", "element": "ELEMENT_SELECTOR", "value": "John Doe" }
```
* The syntax for [ELEMENT_SELECTOR](#ELEMENT_SELECTOR) is defined below.
* Value can also contain:
  * \r   (For "enter key")
  * {SPECIAL_KEY}  (For special keys as defined in the Puppeteer KeyInput object), e.g., {Backspace}
  * {SPECIAL_KEY1}{SPECIAL_KEY2}  (Multiple simultaneous special key presses)
  * List of special keys found here: https://pptr.dev/api/puppeteer.keyinput
* If the input is a checkbox, the value can be:
  * true :: checked
  * false :: unchecked
  * "true" :: checked
  * "false" :: unchecked
* When using variables, the value can be:
  * "@VARIABLE"
  * or contain additional characters, such as "EXAMPLE@VARIABLE\n"

#### Command: Click
The "click" command uses the mouse to interact with the user interface.
```javascript
{ "exec": "click", "element": "ELEMENT_SELECTOR" }
```
* The syntax for [ELEMENT_SELECTOR](#ELEMENT_SELECTOR) is defined below.
* Optional Parameters:
  * "button": "left", "right", "middle"

#### Command: Set
The "set" command saves data from the user interface or page into memory.
```javascript
{ "exec": "set", "variable": "VARIABLE", "value": "VALUE_GETTER" }
```
* The syntax for [VALUE_GETTER](#VALUE_GETTER) is defined below.
* Variables are stored in the jsHarmonyTestConfig.variables object

#### Command: JS
The "js" command can be used to execute arbitray JavaScript.  Promises should be used to wait for the operation to complete before moving to the next command.
```javascript
 { "exec": "js", "js": "return new Promise(function(resolve,reject){ resolve(); });" }
``` 
* Parameters passed to the JS function:
  * jsh - jsHarmony instance
  * page - Puppeteer Page
  * callback - Callback. If a Promise is returned by the JS, wait for the Promise to resolve

#### Command: Assert
The "assert" command checks whether a page element displays an expected value.
```javascript
{ "exec": "assert", "element": "ELEMENT_SELECTOR", "value": "TEXT_COMPARE" }
```
* The syntax for [ELEMENT_SELECTOR](#ELEMENT_SELECTOR) and [TEXT_COMPARE](#TEXT_COMPARE) is defined below.
* Optional Parameters:
  * "error": "Container missing target text"  // Error description

## Selectors and Getters

#### ELEMENT_SELECTOR
Element selectors are used to specify an element in the user interface.
```javascript
  { "selector": "SELECTOR" }  //CSS Selector for an element
  { "selector": "SELECTOR", "visible": true }  //Require element to be visible

  //Example usage:
  { "exec": "wait", "element": {"selector": ".targetElement", "visible": true} },
  
  As a shorthand,
    { "element": "SELECTOR" }
  is equivalent to
    { "element": { "selector": "SELECTOR" } }
```

#### TEXT_COMPARE
Text compare expressions are used to check whether a source string matches an expected value.
```javascript
  { "equals": "text" }       //Text string must be exact match
  { "not_equals": "text" }   //Text string must not match
  { "contains": "text" }     //Text string contains target text
  { "not_contains": "text" } //Text string contains target text
  { "begins_with": "text" }  //Text string begins with target text
  { "ends_with": "text" }    //Text string ends with target text
  { "regex": "text.*" }      //Regex match
  { "equals": "text", "case": "sensitive" } //Default is case sensitive
  { "equals": "text", "case": "insensitive" } //Case insensitive comparison

  //Example usage:
  { "exec": "wait", "text": { "contains": "Target Text" } }

  As a shorthand,
    { "text": "TEXT" }
  is equivalent to
    { "text": { "contains": "TEXT" } }
```

#### VALUE_GETTER
Value getters are used to extract the value of an element property from the user interface.
```javascript
  { "element": “selector”, "property": "text"  }

  //Example usage:
  { "exec": "set", "variable": "Variable name", "value": { "element": ".targetElement", "property": "text" } }
```
* Optional Parameters:
  * "regex": "ID (.*)"  Parse resulting text, and extract first match
* Property can be: 
  * "text", or any element property, ex. "innerHTML", "clientHeight", etc.


## Examples

#### Configuration for testing a login page
```javascript
{
  "id": "login",
  "title": "Login",
  "commands": [
    { "exec": "navigate", "url": "/" },
    //Put app token into a variable
    { "exec": "set", "variable": "APPTOKEN", "value": { "element": ".loginLink", "property": "href", "regex": "app\\_token=(.*)"  } }, 
    
    { "exec": "navigate", "url": "/login?app_token=@APPTOKEN" },
    // Waits for text to render before continuing
    { "exec": "wait", "text": { "contains": "Please enter username" } },
    { "exec": "screenshot", "id": "Login" },

    { "exec": "input", "element": ".username", "value": "testUser" },
    { "exec": "screenshot", "id": "Username" }, 
    { "exec": "click", "element": ".loginButton" },
  ],
}
```

## Tools
#### jsharmony test recorder
The test recorder tool launches a browser and records the user interactions to generate a test script.
```
jsharmony test recorder
```
* Optional flags:
  * --full-element-paths: return full element paths instead of shortest unique path
## Release History

* 1.0.0 Initial release