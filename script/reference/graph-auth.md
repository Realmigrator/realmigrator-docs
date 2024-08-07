# MS Graph Authentication

## User Authentication

| Script                           | Function                                                                                                                                                                                                                     |
| -------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `GraphConnection::isConnected()` | <p>Will test if a user has a valid token.<br><strong>Note</strong>: Script will return <strong>false</strong> if the client is offline. Check internet acces before using any <strong>GraphConnection</strong> features.</p> |
|                                  | If **GraphConnection** returns **true**, `GraphConnection::getUserId()` will return the token principial name.                                                                                                               |

In order to request a new token, you need to proceed the following steps:

1. Start a web browser with the uri returned from `GraphConnection::getAuthorizeUrl(login_hint)`.\
   The login\_hint can be a UPN if you know the user's UPN, or "".
2. Check if the browser is redirected to an uri starting with the string returned by `GraphConnection::getResponseUrl()`.
3. Put the redirect uri from the browser to `GraphConnection::setAuthorizationResponse(string ResponseUrl)`.\
   If the call succeeds, the user received a valid token.

### Authentication sample

```c++
print("start");

class graphAuthentication
{
    def graphAuthentication() {
        this.set_explicit(true);

        this.browser = gkScript.WebBrowser();
        this.graph = gkScript.GraphConnection();

	    if (this.graph.isConnected()) {
            var str = this.graph.getUserId();
            this.startPage = "<html><header><title>Account OK</title></header><body><h1>Account OK</h1><div>Your are logged in as " + str + ".</div><div><a href=\"https://localhost/close\">Close</a></div></body></html>";

			this.responseUrl = "";
        }
		else {
		    var str = this.graph.getAuthorizeUrl(""); // we could provide a login hint here...
            this.startPage = "<html><header><title>Welcome</title></header><body><h1>Welcome</h1><div>You need to login.</div><div><a href=\"" + str + "\">Continue</a></div></body></html>";

			this.responseUrl = this.graph.getResponseUrl();
		}
    };

    def Show() {
        this.browser.Show("about:blank", this);
    }

    def onBeforeNavigate(str) {
        gkScript.logDebug("onNavigate called " + str);

		if (str == "about:blank") {
		    if (!this.startPage.empty()) {
                gkScript.logDebug("setting startPage " + this.startPage);
                this.browser.SetContent(this.startPage);
		    	this.startPage = "";
			    return false;
			}
		}
		else if (str == "https://localhost/close") {
		    this.browser.Close();
			return false;
		}
		else if ((str.size() > this.responseUrl.size()) && (this.responseUrl == str.substr(0, this.responseUrl.size()))) {
            gkScript.logDebug("checking authorization response...");
		    if (this.graph.setAuthorizationResponse(str)) {
			    var user = this.graph.getUserId();
			    gkScript.logInfo("Graph authorisation succeeded, user is: " + user);
                var str = "<html><header><title>Success</title></header><body><h1>Login succeeded</h1><div>Your are logged in as " + user + ".</div><div><a href=\"https://localhost/close\">Close</a></div></body></html>";
                this.browser.SetContent(str);
				return false;
			}
			else {
			    gkScript.logError("Graph authorisation failed. (" + str + ").");
                var str = "<html><header><title>Error</title></header><body><h1>Login failed</h1><div>Please try again later.</div><div><a href=\"https://localhost/close\">Close</a></div></body></html>";
                this.browser.SetContent(str);
				return false;
			}
		}
        gkScript.logDebug("opening target");
        return true;
    }

    var browser;
    var graph;
	var startPage;
	var responseUrl;
}

gkScript.setLogLevel(ll_DEBUG);

var ga = graphAuthentication();
ga.Show();

print("end");

```
