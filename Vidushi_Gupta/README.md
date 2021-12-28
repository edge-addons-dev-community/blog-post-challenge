# Make your first utility Edge Add-on
While surfing sites we usually like the colour palette or the font family of a webpage. What if I tell you that now you can make your own edge add-on to detect the web page design elements to design and develop your next website/app? You can make this beginner friendly extension in just **7 simple** steps. 

## What is an Edge Add-on? 
> A Microsoft Edge extension is a small program that developers use to add or modify features of Microsoft Edge. An extension improves a user's browsing experience. It usually provides a niche function that is important to a target audience. 

In other words, it is similar to a web application which works with a part or the whole of your browser.
You can refer to the official documentation in the [Microsoft Edge Development docs](https://docs.microsoft.com/en-us/microsoft-edge/extensions-chromium/getting-started/)

## Let’s get started on building this add-on:
(Please note the word `extension` and `add-on` have been interchangeably used here and mean the same)

We would be building an add-on that detects the font-family that is used on a part of the webpage. In the very same way, you can also detect the font size, line height, font colour and other web design parameters. 

**Step 1:** Create a new directory which contains the add-on. Let’s name it `Font detection add_on`

**Step 2:** Create a file with the name of [`manifest.json`](https://docs.microsoft.com/en-us/microsoft-edge/extensions-chromium/getting-started/part1-simple-extension).
The manifest JSON file contains basic platform information like the version of the extension, the permissions, title, icon, etc. This is the blueprint of the add-on. 
```json
{
  "manifest_version": 2,
  "name": "Font family detection",
  "version": "1",
  "description": "Helps detect font family",
  "content_scripts": [
    {
      "matches": ["<all_urls>"],
      "css":["popup.css"],
      "js": ["content.js"]
    }
  ],
  "browser_action": {
    "default_icon": "icon.png",
    "default_popup": "popup_font.html"
  },
  "permissions": [
    "activeTab"
  ]

}
```
We start off by specifying the `Manifest version` as `2` as the manifest version 1 is depreciated and we use the version 2 currently. Next, we mention the name of our add-on which is “Font-family detection” in our case. With this we specify the version of this extension. We chose version 1 and then we could further move on to version 1.1, 1.2, 2.1, etc. as we progress with newer features. A description of the add-on is provided next. The `content_scripts` tells the add-on where to access the stylesheets (css) and the js files for functionality. Along with this it also specifies the urls that this function should work on – which in our case is `<all_urls>`. The `browser_action` directs the add-on to the default icon and the default html to be rendered on the browser. In the end, we give the extension permissions – which in our case is the `activeTab`. 

*P.S: Make sure you add an image as “icon.png” for the default icon.*

**Step 3:** Creating a default html file – `popup.html`

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>Font family detection</title>
    <link rel="stylesheet" href="popup.css">
  </head>
  <body>
    <a href="main_popup.html"></a>
    <button type="button" class="font" id="Family">Font Family</button>
  <script type="text/javascript" src="popup.js"></script>
  </body>
</html>
```
In this file we mention the user interface of the extension. In our case we add a button with the name “Font Family” on it. With this we link in the javascript file `popup.js` that tells the function of the add-on. 

**Step 4:** Create the `popup.js` file that mentions the function to be executed by the add-on.

```js
(function user_choice(){
  let buttons = document.querySelectorAll('.font')
  buttons.forEach(item => {item.addEventListener('click', activate);});

  function activate(event){
    let buttons = document.querySelectorAll('.font')
    let chosenId = event.target.attributes.id.nodeValue;
    buttons.forEach(item => {item.classList.remove('active');})
    document.getElementById(chosenId).classList.add('active');

    chrome.tabs.query({active: true, currentWindow: true}, function(tabs){
      chrome.tabs.sendMessage(tabs[0].id, {userChoice: chosenId});
    })
  }
})();
```
We mention the `user_choice` to be executed on the current window (which is the active tab) in the above function. The code in `popup.js` sends a message to your content script that we create and inject into your browser tab.

**Step 5:** Create a `content.js` file which is the content script for your add-on.

```js
function init(){
  createDisplayBox();
  browserListeners();
}

function createDisplayBox(){
  let display_box = document.createElement("div");
  display_box.setAttribute("id", "extension");
  display_box.setAttribute("style",
  `position: absolute;
  background-color: #faea00;
  color: #000000;
  padding: 10px 15px 10px 15px;
  border-radius: 5px;
  font-ramily: Times New Roman;
  font-size: 15px;
  width: auto`);
  document.body.appendChild(display_box);
}

function browserListeners(){
  onmousemove = function(event){
    var box = document.getElementById("extension");
    box.style.left = event.pageX + "px";
    box.style.top = event.pageY + "px";
  }
  chrome.runtime.onMessage.addListener(function(message){
    var choice = message.userChoice;
    if (choice === "Family"){
      onmouseover = function(event){changeText("extension", getCssValue(event, "fontFamily"));}
   }
  })
}

function getCssValue(event, input){
  let elementOver = event.target;
  let text = elementOver.textContent
  let cssObj = window.getComputedStyle(elementOver);
  if(text){return cssObj[input];}
  else{return null}
}

function changeText(id, value){
  document.getElementById(String(id)).textContent = value;
}

init();
```
In the above code, the function `createDisplayBox` gives the style and user interface of the box that displays the font-family of the letters under/near the cursor. The next functions, look for the `fontFamily` in the css and output it in the display box. 

**Step 6:** Create `popup.css` for the styling of the user interface of `popup.html`

```css
button {
  background-color: #faea00;
  color: #000000;
  font-size: 15px;
  width: 160px;
  height: 35px;
  margin: 5px;
  font-family: "Times New Roman";
  border-radius: 5px;
}
.active {
  background-color: #8ce427;
}
body {
  background-color: #0f111b;
}
```
We add basic styling to the popup’s user interface. You can customize the CSS as per your choice. 

**Step 7:** Loading your extension and testing it.

- Head on to `edge://extensions/` on the edge browser.
- Enable the **Developer Mode**
- Click on **Load unpacked** and select your directory (Font detection add_on). Make sure that the `manifest.json` is in the root of the directory. 

![loading extension]()

- **Reload** your extension after you make any changes to the code and you’re good to go. 
- From the extension panel on the toolbar, select your add-on and *hover* on the webpage to see the font-family.

![working of extension]()

Voila! And now you have made your first ever utility edge add-on. Go ahead and enhance this extension further by extracting other CSS features of this webpage!

That’s it for this blog. Keep building and hustling✨
Come say hi on my [socials](https://tinyurl.com/Vidushi-SocialLinks) 👋🏻
