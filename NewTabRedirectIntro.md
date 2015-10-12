# Introduction #

---

## Background ##
(see http://code.google.com/chrome/extensions/overview.html for a thorough background of Chrome Extension development)

A Google Chrome Extension is a mini web application which runs in its own process and can perform actions at a browser-level or page-level.

An extension consists (or may consist) of a number of parts:

  * manifest.json
  * Web files (html, js, css, images)
  * NPAPI Plugins
  * background.html (the long-running process)

All of these files are bundled into a package with a .crx file extension. When uploading your extension to the Google Chrome Extensions Gallery, you only zip up the files your extension needs and Google will package it properly for you.

The **manifest.json** file is a file which specifies properties of your extension in JavaScript Object Notation (aka JSON). For a quick intro to JSON, check out http://www.learn-ajax-tutorial.com/Json.cfm

### The Manifest File ###
manifest.json:
```
{
  "name": "New Tab Redirect!",
  "description": "Sets a user-specified URL to load in new tabs.",
  "version": "0.0.1.109",
  "background_page": "background.html",
  "options_page":"options.html",
  "chrome_url_overrides": {
    "newtab": "redirect.html"
  },
     "permissions": ["tabs"],
     "icons": {
     "128": "icon128.png",
     "19":"icon19.png"
   }
 }
```
(end manifest.json)

The name, description, version, background\_page, and options\_page key/value pairs are pretty self-explanatory.
The "chrome\_url\_overrides" section is interesting. As you can see, the value for this entry is itself an object. This object allows me to override "newtab" with my own page, "redirect.html". Currently, the chromium team will only allow you to override newtab, however, I anticipate they'll allow you to override other pages in the future. For a list of url constants used within Chrome, check out the url\_constants.cc file in the source code repository. Possible future overrides would be anything that is listed near the end of that file as 'const char kChromeUI\*Host[.md](.md)'

Back to the file. "permissions" takes an array of requested permissions. New Tab Redirect! requires permissions to update tabs. This is because we're not only redirecting to html web-hosted sites; New Tab Redirect allows you to set web pages, local files, or inherent Chrome pages.

"icons" specifies the location of the icons your extension will use. These icons will be displayed in the Gallery and in the Extensions page within Google Chrome.

Remember: You must change your version number when updating your publicly hosted extension, or else Chrome won't find a new version and your users will not be updated.

### The Background Page ###
background.html
```
<html>
<head>
<script type="text/javascript">
String.prototype.startsWith = function(str){
    return (this.indexOf(str) === 0);
}

var url = null;
var newTabId = undefined;
 var protocol =  undefined;
 var allowedProtocols = ["http://","https://","about:","file://",
   "file:\\", "file:///", "chrome://","chrome-internal://"];

 function setUrl(url) {
         if(protocolPasses(url)
           && url.length >  { /* 8 is arbitrary */
             this.url = url;
         } else {
             protocol = 'http://'; /* force http */
             var right = url.split('://')
             if(right != undefined && right != null && right.length > 1) {
                 this.url = protocol + right[1];
             } else {
                 /* this will redirect to http:// if url is empty */
                 this.url = protocol + url; }
         }
         localStorage["url"]  = this.url;

     function protocolPasses(url) {
         if(typeof(url) == 'undefined' || url == null) { return false; }
         if (url.startsWith(allowedProtocols[3])
             && !url.startsWith(allowedProtocols[5])) {
             url.replace(allowedProtocols[3], allowedProtocols[5]);
         } else if (url.startsWith(allowedProtocols[4])) {
             url.replace(allowedProtocols[4], allowedProtocols[5]);
         }

         for(var p in allowedProtocols) {
             if(url.startsWith(allowedProtocols[p])) { return true;}
         }
         return false;
     }
 }

 function init() {
    url = localStorage["url"] || "http://www.facebook.com";
 }

 function r(tabId) {
     chrome.tabs.update(tabId, {"url":this.url});
 }

 chrome.tabs.onUpdated.addListener(function(tabId,info,tab) {
     if (info.status === 'loading')
         console.log('Loading url ... ' + tab.url)
     if(info.status === 'complete')
         console.log('Finished loading url ... ' + tab.url)
 });

 chrome.tabs.onCreated.addListener(function(tab) {
     newTabId = tab.id
 });
 </script>
 </head>
 <body onload="init()"></body>
 </html>
```
(end background.html)

This page is pretty self-explanatory for the most part. The background page is your long-running process, and is usually used to house any variables that are shared between pages. However, this isn't a necessity of Chrome Extension architecture. Every page in an extension can interact with any other page through chrome.extension.getViews() or chrome.extension.getBackgroundPage() . Because of the nature of New Tab Redirect, I've used a background page.

One thing of note is the initialization of the extension. Since the operation is fairly simple, allowing a user to specify a url and then redirecting to that url, the only thing I need to worry about immediately is the url. This is stored using HTML 5's local storage. If the url doesn't exit, I set it to facebook.

The setUrl function is located in the Background page and is called from options.html. It is located here so the options page isn't trying to set the url variable that is maintained by the background page. This function checks my array of allowed protocols and validates the url with some simple rules.

r(tabId) is the function which actually updates the tab when the protocol isn't an http or https protocol. The function name is short because the the code in redirect.html should be as concise as possible. This function calls the chrome.tabs.update method (one culprit behind the tabs permission requirement)

That tabId is provided via the redirect.html, which gets the currently created tab's id. However, again note that this processing only occurs when the url is local.

## Options ##
Because the options page is simple html and styles, I'll only include the JavaScript which sets the url.

### options.js ###
```
String.prototype.startsWith = function(str) {
     return (this.indexOf(str)===0);
 }
 var chromePages = {
     Extensions : "chrome://extensions/",
     History : "chrome://history/",
     Downloads : "chrome://downloads/",
     NewTab : "chrome-internal://newtab/"
 }
  var aboutPages = ["about:blank","about:version", "about:plugins","about:cache",
      "about:memory","about:histograms","about:dns",
      "chrome://extensions/","chrome://history/",
      "chrome://downloads/","chrome-internal://newtab/"];
  var popularPages = {
      "Facebook":"www.facebook.com",
      "MySpace":"www.myspace.com",
      "Twitter":"www.twitter.com",
      "Digg":"www.digg.com",
      "Delicious":"www.delicious.com",
      "Slashdot":"www.slashdot.org"
  };

  // save options to localStorage.
  function save_options() {
      var url = $('#custom-url').val();
      if(url == ""){
          url = aboutPages[0];
      }

      if( $.inArray(String(url), aboutPages) || isValidURL(url)) {
          save(true,url);
      } else {
          save(false,url);
      }
  }

  function save(good,url) {
      if(good) {
          chrome.extension.getBackgroundPage().setUrl(url);
          $('#status').text("Options Saved.");
      } else {
          $('#status').text( url + " is invalid. Try again (http:// is required)");
      }

      $('#status').css("display", "block");
      setTimeout(function(){
          $('#status').slideUp("fast").css("display", "none");
      }, 1050);
  }

  // Restores select box state to saved value from localStorage.
  function restore_options() {
      var url = chrome.extension.getBackgroundPage().url || "http://www.facebook.com/";
       $('#custom-url').val(url);
  }

  function isValidURL(url) {
      var urlRegxp = /^(http:\/\/www.|https:\/\/www.|ftp:\/\/www.|www.){1}([\w]+)(.[\w]+){1,2}$/;
      if (urlRegxp.test(url) != true) {
          return false;
      } else {
          return true;
      }
  }

  function saveQuickLink(url){
      var uurl = unescape(url);
       $('#custom-url').val(uurl);
       save(true,uurl);
       return false;
  }

  $(document).ready(function(){
      restore_options();
      $.each(chromePages, function(k,v) {
          var anchor = "<a href=\"javascript:saveQuickLink('"+v+"');\">"+k+"</a>";
          $('#chromes').append("<li>" + anchor + "</li>");
      });
      $.each(aboutPages, function() {
          if(this.startsWith("about:")) { /* quick fix to handle chrome pages elsewhere */
              var anchor = "<a href=\"javascript:saveQuickLink('"+this+"');\">"+this+"</a>";
              $('#abouts').append("<li>" + anchor + "</li>");
          }
      });
      $.each(popularPages, function(k,v) {
          var anchor = "<a href=\"javascript:saveQuickLink('"+v+"');\">"+k+"</a>";
          $('#popular').append("<li>" + anchor + "</li>");
      });
  });

```
(end options.js)

As you can see, I've included jQuery for dom manipulation. I decided to do this because jQuery is so small and because the Options page can be whatever size you'd like (keeping in mind user experience, of course). This saved me some time from writing out a little bit of javascript.

Of note here is how I build the lists of Chrome pages, About pages, and Popular pages. This isn't included in the options.html file, which only has placeholders for the unordered lists. This allows me to go in and provide a page name and url for chromePages and popularPages, or just the url for aboutPages. If I want to add anything in the future, I'll just add a key/value pair (or url) to one of these objects. If I want to remove, likewise, I only touch the respective object.

Quick links (one from an above-mentioned object) are saved automatically, bypassing the url checking regex. However, you'll notice in the save\_options function that I check first to see if the url is in the list of about pages, this is because a user will most likely enter about:memory or any valid url, and whould be less likely to enter chrome-internal://newtab/. In the future, I will probably add checks on all objects before checking a valid url, but this kind of thing happens in incremental development.

As you can see, if the user gets to save the url, the function setUrl from the background page is called via chrome.extension.getBackgroundPage().setUrl(url); Following good user interface techniques, we have to let the user know what happened, which is what the next line and the rest of the function accomplishes.

Note: The new tab url is chrome-internal:// instead of chrome://. Why is this, do you think? Don't think too hard, though. It's simple: redirect.html is now chrome://newtab/, because I've told chrome to override the newtab with my own file. You can test it out by typing chrome://newtab into Google Chrome. Try chrome-internal://newtab and see what happens? This can only be called internally, hence the name, and must be called from within an extension via chrome.tabs.update. If you don't use this internal url, when you set the url to chrome://newtab and hit CTRL+T or click the '+' tab, your tab will keep trying to call itself, and eventually stop, after doing nothing meaningful.

## Redirect ##
Finally, the beast that does it all.

### redirect.html ###
```
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
<html>
  <head>
    <meta name="generator" content=
    "HTML Tidy for Windows (vers 14 February 2006), see www.w3.org">
    <title>
      Redirecting...
    </title>
    <script type="text/javascript">
     var wid, tid;

     function r() {
         var url = localStorage["url"] || "";
         if (url.toLowerCase().indexOf("http:") === 0 ||
             url.toLowerCase().indexOf("https:") === 0) {
             document.location.href = url;
             return;
         } else {
             chrome.windows.getCurrent(function(w) {
                 wid = w.id;
                 chrome.tabs.getSelected(wid, function(t) {
                     tid = t.id;
                 });
             });
             chrome.extension.getBackgroundPage().r(tid);
         }
     }
 </script>
   </head>
   <body onload="setTimeout('r()',100);r();return false;">
     Redirecting...<br>
     <em>If your page doesn't load within 5 seconds,
       <a href="javascript:r()">click here</a></em>
   </body>
 </html>
```
(end redirect.html)

Note: I have run the redirect.html file through HTML Tidy and a JavaScript formatter at http://jsbeautifier.org/, but this file should be as compact as possible. If you were to download New Tab Redirect and look at the source code, you'd see this code is all on one line. I'm writing this for your enjoyment, though, and one-liners aren't fun to look at.

The only thing special about this file is the the onload function. You'll notice how it runs r after 100 ms, then runs it again. There is quite an odd situation here: Chrome either has to initialize localStorage, doesn't allow us to get the current window and tab id immediately, or the processing required to do so takes longer than 100ms. You could, honestly, run the redirect function through a loop until it redirects (considering the document.location.href will immediately redirect the browser tab. If the url specified by the user is local, the function gets the current window, and from that the id of the current tab, and passes that to the background page's function.

I've been asked by a user or two to remove the redirect message, but I'll keep it for now. I consider a message necessary if, for some reason, a future release of Chrome breaks the extension.

The current source code for New Tab Redirect can be accessed via your Chrome Extensions directory after installing the extension, and is availing at the New Tab Redirect Project Page on Google Code.

If you have any questions or comments about this extension, please contact me.