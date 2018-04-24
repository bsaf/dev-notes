# Everything I know about making Chrome Extensions

## Basic setup

You will need a manifest.json, a hello.html, and an icon as a png.

manifest.json:

```
{
  "name": "Hello Extensions",
  "description": "A simple extension.",
  "version": "1.0",
  "manifest_version": 2
}
```

Add a browser_action item in the manifest.json to open a popup when you click the extension.

```
  "browser_action": {
    "default_popup": "hello.html",
    "default_icon": "hello_extensions.png"
  }
```

Add a "commands" item to set a keyboard shortcut:

```
  "commands": {
    "_execute_browser_action": {
      "suggested_key": {
        "default": "Ctrl+Shift+F",
        "mac": "MacCtrl+Shift+F"
      },
      "description": "Opens hello.html"
    }
  }
```

## Adding a background script

A background script is Javascript that ...

Create background.js and then include the script via the manifest.json:

```
  "background": {
    "scripts": ["background.js"],
    "persistent": false
  },
```

## Running code on installation

To do this, listen for runtime.onInstalled within the background script.

```
  chrome.runtime.onInstalled.addListener(function() {
    // do something
  });
```

## Storing something using the storage API

Use storage.sync.set(data, callback):

```
  chrome.storage.sync.set({color: '#3aa757'}, function() {
    console.log("The color is green.");
  });
```

You'll also need to register the "storage" permission in the manifest.json:

```
  "permissions": ["storage"]
```

## Getting something out of storage

First, make sure you've added the storage permission to the manifest.json (see above).

Then, get things out like this:

```
chrome.storage.sync.get('color', function (data) {
  changeColor.style.backgroundColor = data.color;
  changeColor.setAttribute('value', data.color);
});
```

## Displaying a popup when you click the extension

You can use browser_action in the manifest.json -- this will make the extension active at all times.

```
  "browser_action": {
    "default_popup": "hello.html"
  }
```

If you only want the popup to be active on specific pages, use page_action in manifest.json:

```
  "page_action": {
    "default_popup": "hello.html"
  }
```

And you also need to add declared rules using the declaritiveContent API. This one is saying "only let this work on developer.chrome.com"). We do this inside the runtime.onInstalled listener event:

```
chrome.runtime.onInstalled.addListener(function () {

  chrome.declarativeContent.onPageChanged.removeRules(undefined, function () {
    chrome.declarativeContent.onPageChanged.addRules([{
      conditions: [new chrome.declarativeContent.PageStateMatcher({
        pageUrl: { hostEquals: 'developer.chrome.com' },
      })
      ],
      actions: [new chrome.declarativeContent.ShowPageAction()]
    }]);
  });

});
```

Finally, you need to add permission to access the `declarativeContent` API in the `manifest.json`.

```
    "permissions": ["declarativeContent"],
```

## Displaying an icon

Toolbar icons – add `default_icon` to `page_action` or `browser_action`:

```
    "page_action": {
      "default_icon": {
        "16": "images/get_started16.png",
        "32": "images/get_started32.png",
        "48": "images/get_started48.png",
        "128": "images/get_started128.png"
      }
    },
```

Other icons (extension management page, the permissions warning, and favicon) – add to an "icons" item – not in `page_action`:

```
    "icons": {
      "16": "images/get_started16.png",
      "32": "images/get_started32.png",
      "48": "images/get_started48.png",
      "128": "images/get_started128.png"
    },
```

## Adding external script file to your popup

Just include normally:

```
<script src="popup.js"></script>
```

In that script, you can call the Chrome APIs:

```
let changeColor = document.getElementById('changeColor');

chrome.storage.sync.get('color', function (data) {
  changeColor.style.backgroundColor = data.color;
  changeColor.setAttribute('value', data.color);
});
```

## Running code on a web page

You can **programatically inject** code into the active tab – so you could inject the code when a button is clicked.

The `chrome.tabs.query()` returns the active tab.

Then the `chrome.tabs.executeScript()` executes the code against a specific tab id.

```
  changeColor.onclick = function(element) {
    let color = element.target.value;
    chrome.tabs.query({active: true, currentWindow: true}, function(tabs) {
      chrome.tabs.executeScript(
          tabs[0].id,
          {code: 'document.body.style.backgroundColor = "' + color + '";'});
    });
  };
```

You will also need to give the extension access to the tabs API:

```
    "permissions": ["activeTab"]
```

## Adding an options page

First, add `options.html` to the project. Then register the options page in the manifest.json root object:

```
"options_page": "options.html",
```

You can get to this page by reloading the extension, clicking DETAILS, then 'Extension options'.

You can reference a .js file from the options page. This one sets a clicked colour into chrome.storage:

```
const kButtonColors = ['#3aa757', '#e8453c', '#f9bb2d', '#4688f1']
const page = document.body
function constructOptions(kButtonColors) {
  for (let item of kButtonColors) {
    let button = document.createElement('button');
    button.style.backgroundColor = item;
    button.addEventListener('click', function () {
      chrome.storage.sync.set({ color: item }, function () {
        console.log('color is ' + item);
      })
    });
    page.appendChild(button);
  }
}
constructOptions(kButtonColors);
```






