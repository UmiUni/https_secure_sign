https is for adding the following ReferHelp app config file for universal link.
# https_secure_sign

```
{
    "applinks": {
        "apps": [],
        "details": [
            {
                "appID": "CR32K9WG2A.com.umiuni.ReferHelper",
                "paths": [ "/signup", "/reset" ]
            }
        ]
    }
}
```

```https://<domain>/apple-app-site-association```
or
```https://<domain>/.well-known/apple-app-site-association```

```https://referhelper.com/.well-known/apple-app-site-association```

https://developer.apple.com/library/archive/documentation/General/Conceptual/AppSearch/UniversalLinks.html


Support Universal Links
When you support universal links, iOS users can tap a link to your website and get seamlessly redirected to your installed app without going through Safari. If your app isn’t installed, tapping a link to your website opens your website in Safari.

Universal links give you several key benefits that you don’t get when you use custom URL schemes. Specifically, universal links are:

Unique. Unlike custom URL schemes, universal links can’t be claimed by other apps, because they use standard HTTP or HTTPS links to your website.

Secure. When users install your app, iOS checks a file that you’ve uploaded to your web server to make sure that your website allows your app to open URLs on its behalf. Only you can create and upload this file, so the association of your website with your app is secure.

Flexible. Universal links work even when your app is not installed. When your app isn’t installed, tapping a link to your website opens the content in Safari, as users expect.

Simple. One URL works for both your website and your app.

Private. Other apps can communicate with your app without needing to know whether your app is installed.

NOTE

Universal links let users open your app when they tap links to your website within WKWebView and UIWebView views and Safari pages, in addition to links that result in a call to openURL:, such as those that occur in Mail, Messages, and other apps.

When a user is browsing your website in Safari and they tap a universal link to a URL in the same domain as the current webpage, iOS respects the user’s most likely intent and opens the link in Safari. If the user taps a universal link to a URL in a different domain, iOS opens the link in your app.

For users who are running versions of iOS earlier than 9.0, tapping a universal link to your website opens the link in Safari.

Adding support for universal links is easy. There are three steps you need to take:

Create an apple-app-site-association file that contains JSON data about the URLs that your app can handle.

Upload the apple-app-site-association file to your HTTPS web server. You can place the file at the root of your server or in the .well-known subdirectory.

Prepare your app to handle universal links.

You can test universal links on a device.

Creating and Uploading the Association File
To create a secure connection between your website and your app, you establish a trust relationship between them. You establish this relationship in two parts:

An apple-app-site-association file that you add to your website

A com.apple.developer.associated-domains entitlement that you add to your app (this part is described in Preparing Your App to Handle Universal Links)

You can learn more about how your app and website can share credentials in Shared Web Credentials Reference.

NOTE

If your app runs in iOS 9 or later and you use HTTPS to serve the apple-app-site-association file, you can create a plain text file that uses the application/json MIME type and you don’t need to sign it. If you support Handoff and Shared Web Credentials in iOS 8, you still need to sign the file as described in Shared Web Credentials Reference.

You need to supply a separate apple-app-site-association file for each domain with unique content that your app supports. For example, apple.com and developer.apple.com need separate apple-app-site-association files, because these domains serve different content. In contrast, apple.com and www.apple.com don’t need separate site association files—because both domains serve the same content—but both domains must make the file available. For apps that run in iOS 9.3.1 and later, the uncompressed size of the apple-app-site-association file must be no greater than 128 KB, regardless of whether the file is signed.

In your apple-app-site-association file, you specify the paths from your website that should be handled as universal links along with those that should not be handled as universal links. Keep the list of paths fairly short and rely on wildcard matching to match larger sets of paths. Listing 6-1 shows an example of an apple-app-site-association file that identifies three paths that should be handled as universal links.

Listing 6-1Creating an apple-app-site-association file
{
    "applinks": {
        "apps": [],
        "details": [
            {
                "appID": "9JA89QQLNQ.com.apple.wwdc",
                "paths": [ "/wwdc/news/", "/videos/wwdc/2015/*"]
            },
            {
                "appID": "ABCD1234.com.apple.wwdc",
                "paths": [ "*" ]
            }
        ]
    }
}
NOTE

Don’t append .json to the apple-app-site-association filename.

The apps key in an apple-app-site-association file must be present and its value must be an empty array, as shown in Listing 6-1. The value of the details key is an array of dictionaries, one dictionary per app that your website supports. The order of the dictionaries in the array determines the order the system follows when looking for a match, so you can specify an app to handle a particular part of your website.

Each app-specific dictionary contains an appID key and a paths key. The value of the appID key is the team ID or app ID prefix, followed by the bundle ID. (The appID value is the same value that’s associated with the “application-identifier” key in your app’s entitlements after you build it.) The value of the paths key is an array of strings that specify the parts of your website that are supported by the app and the parts of your website that you don’t want to associate with the app. To specify an area that should not be handled as a universal link, add “NOT ” (including a space after the T) to the beginning of the path string. For example, the apple-app-site-association file shown in Listing 6-1 could prevent the /videos/wwdc/2010/* area of the website from being handled as a universal link by updating the paths array as shown here:

"paths": [ "/wwdc/news/", "NOT /videos/wwdc/2010/*", "/videos/wwdc/201?/*"]
Because the system evaluates each path in the paths array in the order it is specified—and stops evaluating when a positive or negative match is found—you should specify high priority paths before low priority paths. Note that only the path component of the URL is used for comparison. Other components, such as the query string or fragment identifier, are ignored.

There are various ways to specify website paths in the apple-app-site-association file. For example, you can:

Use * to specify your entire website

Include a specific URL, such as /wwdc/news/, to specify a particular link

Append * to a specific URL, such as /videos/wwdc/2015/*, to specify a section of your website

In addition to using * to match any substring, you can also use ? to match any single character. You can combine both wildcards in a single path, such as /foo/*/bar/201?/mypage.

NOTE

The strings you use to specify website paths in the paths array are case sensitive.

After you create the apple-app-site-association file, upload it to the root of your HTTPS web server or to the .well-known subdirectory. The file needs to be accessible via HTTPS—without any redirects—at https://<domain>/apple-app-site-association or https://<domain>/.well-known/apple-app-site-association. Next, you need to handle universal links in your app.

Preparing Your App to Handle Universal Links
Universal links use two technologies: The first is the same mechanism that powers Handoff between a web browser and a native app, and the second is Shared Web Credentials (for more information about these technologies, see Web Browser–to–Native App Handoff and Shared Web Credentials Reference). When a user taps a universal link, iOS launches your app and sends it an NSUserActivity object that you can query to find out how your app was launched.

To support universal links in your app, take the following steps:

Add an entitlement that specifies the domains your app supports.

Update your app delegate to respond appropriately when it receives the NSUserActivity object.

In your com.apple.developer.associated-domains entitlement, include a list of the domains that your app wants to handle as universal links. To do this in Xcode, open the Associated Domains section in the Capabilities tab and add an entry for each domain that your app supports, prefixed with applinks:, such as applinks:www.mywebsite.com. Limit this list to no more than about 20 to 30 domains.

To match all subdomains of an associated domain, you can specify a wildcard by prefixing *. before the beginning of a specific domain (the period is required). Domain matching is based on the longest substring in the applinks entries. For example, if you specify the entries applinks:*.mywebsite.com and applinks:*.users.mywebsite.com, matching for the domain emily.users.mywebsite.com is performed against the longer *.users.mywebsite.com entry. Note that an entry for *.mywebsite.com does not match mywebsite.com because of the period after the asterisk. To enable matching for both *.mywebsite.com and mywebsite.com, you need to provide a separate applinks entry for each.

After you specify your associated domains, adopt the UIApplicationDelegate methods for Handoff (specifically application:continueUserActivity:restorationHandler:) so that your app can receive a link and handle it appropriately.

When iOS launches your app after a user taps a universal link, you receive an NSUserActivity object with an activityType value of NSUserActivityTypeBrowsingWeb. The activity object’s webpageURL property contains the URL that the user is accessing. The webpage URL property always contains an HTTP or HTTPS URL, and you can use NSURLComponents APIs to manipulate the components of the URL.

When a user taps a universal link that you handle, iOS also examines the user’s recent choices to determine whether to open your app or your website. For example, a user who has tapped a universal link to open your app can later choose to open your website in Safari by tapping a breadcrumb button in the status bar. After the user makes this choice, iOS continues to open your website in Safari until the user chooses to open your app by tapping OPEN in the Smart App Banner on the webpage.

NOTE

If you instantiate a SFSafariViewController, WKWebView, or UIWebView object to handle a universal link, iOS opens your website in Safari instead of opening your app. However, if the user taps a universal link from within an embedded SFSafariViewController, WKWebView, or UIWebView object, iOS opens your app.

It’s important to understand that if your app uses openURL: to open a universal link to your website, the link does not open in your app. In this scenario, iOS recognizes that the call originates from your app and therefore should not be handled as a universal link by your app.

If you receive an invalid URL in an activity object, it’s important to fail gracefully. To handle an unsupported URL, you can call openURL: on the shared application object to open the link in Safari. If you can’t make this call, display an error message to the user that explains what went wrong.

IMPORTANT

To protect users’ privacy and security, you should not use HTTP when you need to transport data; instead, use a secure transport protocol such as HTTPS.
