# Web Browser

## Simple Web Browser

| Script                      | Function                                                                                                                  |
| --------------------------- | ------------------------------------------------------------------------------------------------------------------------- |
| `gkScript.showBrowser(url)` | <p>Starts a web browser object and navigates to an entered URL.<br>The method will return when the browser is closed.</p> |

A simple web browser sample:

```c++
print("start");

gkScript.showBrowser("http://www.google.de")

print("end");
```

## Web Browser Object

| Script                  | Function                                |
| ----------------------- | --------------------------------------- |
| `gkScript.WebBrowser()` | Returns a web browser object            |
| `WebBrowser::Show`      | Defines a callback object and submit it |

Web browser object example:

```c++
class WebBrowser
{
    void Show(uri, callbackObject, bool modal = true);
    void Show(uri, callbackObject, integer flags);
    bool IsVisible();       // true if browser is currently shown
    uint64_t GetWindowHandle();
    void Close();
    void SetFullScreen(bool fullScreen);
    bool SetContent(string HTML);
    bool Navigate(string uri);
    bool EvalJavaScript(string code);

    // flags
    interger showFullScreen;    // show full screen browser. cannot be combined with showFlash
    integer showFlash;          // show flash-style browser.
    integer showFrameless;      // no frame, titlebar and close button on browser window
    integer showModal;          // Show() returns when browser is closed.
}
```

Callback object and submit:

```c++
class callbackObject
{
    bool onBeforeNavigate(string url);
    void onDocumentComplete(string url);
    void onBrowserCallback(string data);
}
```

Web browser sample - callback object:

```c++
class browserController
{
    def browserController() {
       this.browser = gkScript.WebBrowser();
    };

    def Show(url) {
        this.browser.Show(url, this);
    }

    def ShowFlash(url) {
        var flags = gkScript.WebBrowser.showFlash | gkScript.WebBrowser.showFrameless | gkScript.WebBrowser.showModal;
        this.browser.Show(url, this, flags);
    }

    def onBeforeNavigate(str) {
        gkScript.logDebug("onNavigate called " + str);
        if (str.find("cat") != -1) {
            gkScript.logError("cat alert " + str);
            this.browser.SetContent("<html><header><title>Cats detected</title></header><body><h1>Shame on you!</h1><div>We catched you seraching for cats.</div></body></html>");
            return false;
        }
        else if ((str.find("coke") != -1) || (str.find("cola") != -1)) {
            gkScript.logInfo("coke redirection " + str);
            this.browser.Navigate("http://www.pepsi.com");
            return false;
        }
        return true;
    }

    def onBrowserCallback(str) {}

    var browser;

}

var c = browserController();
c.Show("http://www.google.de");
c.ShowFlash("http://www.google.de");
```
