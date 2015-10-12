# Introduction #

There are a few questions which people regularly ask.  This is probably the best place to find an answer.


## Why doesn't the URL bar focus? ##

For security reasons, Google Chrome does not allow extensions to access certain parts of the browser.  This has nothing to do with New Tab Redirect, and has everything to do with Chrome itself.

It is possible to create a static new tab page (i.e. a local or hosted page which can't be changed).  In this case, the new tab will gain focus because this is handled internally in the browser.  Because New Tab Redirect allows the user to dynamically change and load a customized URL, which is done after the browser would focus the address bar, this functionality is lost.  Unfortunately, this the only way to have a customized new tab page which can be easily modified by the user.

If you'd like to create your own static new tab page, you can download [one of the samples](http://src.chromium.org/viewvc/chrome/trunk/src/chrome/common/extensions/docs/examples/api/override/blank_ntp/) from http://code.google.com/chrome/extensions/samples.html and modify it to meet your needs.

There is an experimental API for **interaction** with the omnibar.  However, this interaction is currently little more than watching the omnibar for changes.  It does not allow an extension to modify the bar in anyway.  If/when this functionality is available, I will implement it as soon as possible.

For those who want a quick and dirty way to highlight the address bar, do as I do and click CTRL+L.  This will highlight the omnibar and is only difficult if you don't regularly type with both hands on home row.

## How do I change the new tab page after I have installed? ##

Accessing the options page can be done as with any other extension.

Steps can be found here:
http://www.google.com/support/chrome/bin/answer.py?hl=en&answer=187443

If you still have trouble finding the options page, you may copy/paste one of the following links into your address bar and hit ENTER:
  * All extensions: chrome://extensions/
  * New Tab Redirect options: chrome-extension://icpgjfneehieebagbmdbhnlpiopdcmna/options.html

However, I recommend becoming familiar with accessing the options pages for extensions you decide to install.  Although the Chrome team has done a fairly good job of designing an extension system which provides a great deal of security, there are still extension out there which can track and maintain your personal information without your consent.  This, of course, violates the terms which all developers must accept before submitting an extension to the gallery, but that doesn't stop certain people from breaking the rules.  Gaining the trust of users is one reason why I've hosted the code for the extension.

## How do I edit the extension's source locally? ##

This is a question I've only received once so it doesn't exactly fit the "Frequently Asked" requirement to make this page.  I think this is very important for everyone to know, so I'm including it anyway.

To edit the extension's source code (e.g. options.js), the locations are:

Linux:
~/.config/google-chrome/Default/Extensions/icpgjfneehieebagbmdbhnlpiopdcmna/1.0.0.38\_0/options.js

Windows:
C:\Users\**username**\AppData\Local\Google\Chrome\User Data\Default\Extensions\icpgjfneehieebagbmdbhnlpiopdcmna\1.0.0.38\_0\options.js

Mac:
Macintosh HD\Users\**username**\Library\Application Support\Google\Chrome\Default\Extensions\icpgjfneehieebagbmdbhnlpiopdcmna\1.0.0.38\options.js

Of course, you would need to modify "1.0.0.38" with whatever version you have installed.  Also, different versions of Chrome may maintain the version numbers differently.  Once you are in the folder which begins with "icp..." you should be able to enter the folder with the highest version number to make any local changes.