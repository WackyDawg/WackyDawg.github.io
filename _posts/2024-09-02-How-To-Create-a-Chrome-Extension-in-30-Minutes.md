--- 
title: "How To Create a Chrome Extension in 30 Minutes: A Beginner’s Step By Step Guide"

excerpt: "Chrome extensions are powerful tools that enhance your browsing experience—from blocking ads to automating tasks. But have you ever wondered how they work? In this guide, you’ll bui...."

categories:
- Javascript

tags:
- REST API
- Node.js
- Express
- MongoDB
- API Development
- Backend

--- 

**How To Create a Chrome Extension in 30 Minutes: A Beginner’s Step By Step Guide**  
*Build a Simple Ad-Blocker Step-by-Step*  

---

Chrome extensions are powerful tools that enhance your browsing experience—from blocking ads to automating tasks. But have you ever wondered how they work? In this guide, you’ll build a simple ad-blocking Chrome extension from scratch in **30 minutes**, even if you’ve never coded an extension before. No prior experience required!  

By the end, you’ll understand:  
- The anatomy of a Chrome extension (manifest, scripts, and assets).  
- How to intercept and hide webpage elements like ads.  
- How to test and debug your extension in Chrome.  

Let’s dive in!  

---

## Tools You’ll Need  
1. **Google Chrome**: Download [here](https://www.google.com/chrome/).  
2. A **text editor** (VS Code, Sublime Text, or Notepad++) but i would be using Vscode so choose whatever code editor you prefer.  
3. You must have a basic knowledge of HTML, CSS, and JavaScript since thats what we would be working with.  

---

## Step 1: Set Up Your Project  
Create a folder named `ad-blocker` with the following files:  
```
ad-blocker/  
├── manifest.json  
├── content.js  
└── icons/  
    ├── icon16.png  
    ├── icon48.png  
    └── icon128.png  
```  
- **`manifest.json`**: The extension’s configuration file.  
- **`content.js`**: The script that interacts with webpages.  
- **`icons/`**: Add placeholder icons (or use [these samples](https://placehold.co/)).  

---

## Step 2: Configure the Manifest  
Open `manifest.json` and paste this code:  
```json  
{  
  "manifest_version": 3,  
  "name": "Simple Ad-Blocker",  
  "version": "1.0",  
  "description": "Hides ads from webpages.",  
  "icons": {  
    "16": "icons/icon16.png",  
    "48": "icons/icon48.png",  
    "128": "icons/icon128.png"  
  },  
  "content_scripts": [  
    {  
      "matches": ["<all_urls>"],  
      "js": ["content.js"],  
      "run_at": "document_end"  
    }  
  ],  
  "permissions": ["activeTab"]  
}  
```  

### Key Details:  
- **`manifest_version`**: Always use 3 (Chrome’s latest standard).  
- **`content_scripts`**: Instructs Chrome to inject `content.js` into all URLs.  
- **`run_at`**: Ensures the script runs after the page loads.  

---

## Step 3: Write the Content Script  
Open `content.js` and add this code to hide ad elements:  
```javascript  
// Target common ad-related classes and IDs  
const adSelectors = [  
  '.ad', '.ads', '.ad-banner',  
  '#ad', '#ad-container',  
  'div[data-ad]', 'iframe[src*="ads"]'  
];  

function hideAds() {  
  adSelectors.forEach(selector => {  
    document.querySelectorAll(selector).forEach(element => {  
      element.style.display = 'none';  
    });  
  });  
}  

// Run when the page finishes loading  
window.addEventListener('load', hideAds);  

// Optional: Watch for dynamic ads added after load  
const observer = new MutationObserver(hideAds);  
observer.observe(document.body, {  
  childList: true,  
  subtree: true  
});  
```  

### How It Works:  
1. **`adSelectors`**: A list of CSS selectors targeting common ad containers.  
2. **`hideAds()`**: Hides elements matching the selectors by setting `display: none`.  
3. **MutationObserver**: Watches for new elements (e.g., dynamically loaded ads) and reapplies `hideAds()`.  

---

## Step 4: Add Icon Placeholders  
For testing, create simple 16x16, 48x48, and 128x128 pixel PNGs in the `icons` folder. Name them `icon16.png`, `icon48.png`, and `icon128.png`.  

---

## Step 5: Load the Extension in Chrome  
1. Open Chrome and navigate to `chrome://extensions/`.  
2. Enable **Developer Mode** (toggle in the top-right corner).  
3. Click **Load unpacked** and select your `ad-blocker` folder.  

Your extension will appear in the Chrome toolbar with the placeholder icon.  

---

## Step 6: Test Your Ad-Blocker  
1. Visit a test page with ads (e.g., [this sample page](https://adstestpage.com/)).  
2. Alternatively, create a local HTML file:  
```html  
<!DOCTYPE html>  
<html>  
<body>  
  <div class="ad">Fake Ad 1</div>  
  <div id="ad-container">Fake Ad 2</div>  
  <iframe src="https://example.com/ads.html"></iframe>  
</body>  
</html>  
```  
3. Open the page. The ad elements should be hidden!  

---

## How It Works Under the Hood  
- **Content Scripts**: Run in the context of webpages, allowing DOM manipulation.  
- **Manifest V3**: Prioritizes security and performance over V2.  
- **MutationObserver**: Monitors DOM changes to handle dynamic content (e.g., ads loaded via AJAX).  

---

## Troubleshooting  
**Issue**: Ads aren’t being hidden.  
- **Fix**:  
  - Ensure the `content_scripts` matches include the tested URL.  
  - Check for typos in `content.js` (use Chrome’s DevTools Console: `Ctrl+Shift+J`).  

**Issue**: Extension doesn’t load.  
- **Fix**:  
  - Validate JSON syntax in `manifest.json` (use [JSONLint](https://jsonlint.com/)).  
  - Reload the extension on `chrome://extensions/`.  

---

## Enhance Your Extension (Optional)  
### 1. Block Ads via Network Requests  
Upgrade to request blocking using Manifest V3’s `declarativeNetRequest` API:  
1. Update `manifest.json`:  
```json  
{  
  ...  
  "permissions": ["declarativeNetRequest"],  
  "host_permissions": ["<all_urls>"],  
  "background": {  
    "service_worker": "background.js"  
  }  
}  
```  
2. Create `background.js`:  
```javascript  
chrome.declarativeNetRequest.updateDynamicRules({  
  addRules: [{  
    "id": 1,  
    "priority": 1,  
    "action": { "type": "block" },  
    "condition": {  
      "urlFilter": '||ads.example.com^',  
      "resourceTypes": ['script', 'image']  
    }  
  }]  
});  
```  

### 2. Add a Popup Interface  
Let users customize blocked selectors:  
1. Add `popup.html` and `popup.js`.  
2. Update `manifest.json`:  
```json  
{  
  ...  
  "action": {  
    "default_popup": "popup.html"  
  }  
}  
```  

---

## Conclusion  
Congratulations! You’ve built a functional Chrome extension in under 30 minutes. While this ad-blocker is basic, it demonstrates the core concepts of Chrome extension development:  

1. **Manifest Configuration**: The backbone of your extension.  
2. **Content Scripts**: Interact with webpage content.  
3. **Testing/Debugging**: Use Chrome’s built-in tools.  

**Next Steps**:  
- Explore Chrome’s [Extension API](https://developer.chrome.com/docs/extensions/reference/).  
- Publish your extension to the [Chrome Web Store](https://chrome.google.com/webstore/category/extensions).  

---

**Resources**:  
- [Chrome Extension Documentation](https://developer.chrome.com/docs/extensions/)  
- [Sample Ad-Blocker Code](https://github.com/wackydawg)  
- [Manifest V3 Migration Guide](https://developer.chrome.com/docs/extensions/mv3/intro/)  

**Total Time Spent**: 30 minutes  

--- 

Now go forth and build—your browser awaits customization! 🚀