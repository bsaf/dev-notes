# Everything I know about making Chrome Extensions

## Basic setup

Create an empty folder. In it, you will need a `manifest.json`.


```json
{
  "name": "Hello Extensions",
  "description": "A simple extension.",
  "version": "1.0",
  "manifest_version": 3
}
```

Add a `browser_action` item in `manifest.json` to open a HTML file as a popup when you click the extension. (Create `hello.html` as well.)

```
  "browser_action": {
    "default_popup": "hello.html"
  }
```

## Adding a background script

A background script is Javascript that runs when your extension is loaded. You can put listeners in it to respond to various events.

Create `background.js` and then include the script in the `manifest.json`:

```
  "background": {
    "scripts": ["background.js"],
    "persistent": false
  },
```

## Running code on installation

To run code when the extension is first installed, listen for `runtime.onInstalled` within the background script.

```
  chrome.runtime.onInstalled.addListener(function() {
    // do something
  });
```

## Storing data using the storage API

Use `storage.sync.set(data, callback)` :

```
  chrome.storage.sync.set({color: '#3aa757'}, function() {
    console.log("The color is green.");
  });
```

You'll also need to register the "storage" permission in the manifest.json:

```
  "permissions": ["storage"]
```

See the [API docs](https://developer.chrome.com/apps/storage) for `chrome.storage`.

## Getting something out of storage

First, make sure you've added the storage permission to the `manifest.json` (see above).

Then, get things out like this:

```
chrome.storage.sync.get('color', function (data) {
  changeColor.style.backgroundColor = data.color;
  changeColor.setAttribute('value', data.color);
});
```

## Displaying a popup when you click the extension

You can use `browser_action` in the manifest.json -- this will make the extension active at all times.

```
  "browser_action": {
    "default_popup": "hello.html"
  }
```

If you only want the extension/popup to be active on specific pages, use `page_action` in `manifest.json`:

```
  "page_action": {
    "default_popup": "hello.html"
  }
```

And you also need to add declared rules using the declaritiveContent API. The following example is saying "only let this work on developer.chrome.com"). We do this inside the `runtime.onInstalled` listener event:

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
    "permissions": ["declarativeContent"]
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

Other icons (extension management page, the permissions warning, and favicon) – add to an `icons` item – not in `page_action`, but in the root object:

```
    "icons": {
      "16": "images/get_started16.png",
      "32": "images/get_started32.png",
      "48": "images/get_started48.png",
      "128": "images/get_started128.png"
    },
```

## Adding an external script file to your popup

Just include it normally:

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

## Running your code on a web page

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

First, add `options.html` to the project. Then register the options page in the `manifest.json` root object:

```
"options_page": "options.html",
```

You can get to this page by reloading the extension, clicking DETAILS, then 'Extension options'.

You can reference a .js file from the options page. This one sets a clicked colour into `chrome.storage`:

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

## Adding a keyboard shortcut to invoke the extension

Add a `commands` item in `manifest.json` to set a keyboard shortcut:

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

## Triggering an action when a tab changes (e.g. a hash URL changes)

Check in `manifest.json` to see if you have a background script, or add one.

```
  "background": {
    "scripts": ["background.js"]
  }
```

You also need to add a `tabs` permission to `manifest.json`:

```
  "permissions": ["activeTab", "tabs", "storage"]
```

In the `background.js` (or whatever background script you are using), send a message on `tabs.onUpdated`:

```
chrome.tabs.onUpdated.addListener((tabId, changeInfo, tab) => {
  chrome.tabs.sendMessage(tabId, {
    message: "urlChanged",
    url: tab.url
  });
});
```

In your foreground script, listen for this message:

```
// there's a message!
chrome.runtime.onMessage.addListener((request, sender, sendResponse) => {
  // it's a url changed message!
  if (request.message === "urlChanged") {
    console.log(request)
  }
});
```
