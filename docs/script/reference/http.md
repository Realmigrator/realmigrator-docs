# HTTP Reference

## Http transfer

| Script                                                                                 | Function                    |
| -------------------------------------------------------------------------------------- | --------------------------- |
| `bool gkScript::fetchUrl(string url, string &response, map<string string>header = [])` | Fetches a URL into a string |

Samples:

```c++
var data = "";
gkScript.fetchUrl("https://www.dropbox.com/", data);
gkScript.fetchUrl("https://www.dropbox.com/", data, ["Accept-Charset": "utf-8", "Accept-Language": "en-US"]);
```

## HttpResponse Object

| Script                                                                                                                      | Function                     |
| --------------------------------------------------------------------------------------------------------------------------- | ---------------------------- |
| `HttpResponse gkScript::httpRequest(string url, method httpMethod, map<string string>header = [], string requestBody = "")` | Creates a HttpResponse class |

The following methods are defined:

```c++
enum http_methods    // supported methods for gkScript::httpRequest
{
    GET,
    POST,
    HEAD,
    PUT,
    PATCH,
    DELETE,
    TRACE,
    OPTIONS,
    CONNECT
};
```

### HttpResponse class

```c++
class HttpResponse
{
    statusCode; // unsigned int
    responseBody; // string
    responseHeader; // map of string:string pairs
    string getHeader(string header); // get response-header with case insensitive header name
};
```

### Http transfer sample

The following sample is a combination of the script from **Http transfer** and **HttpResponse Object**. It uses both to get the headers of an URL.\
Then a **GET** method with range header to retrieve the first 100 bytes of the URL.

```shell
eval> var response=gkScript.httpRequest("http://www.glueckkanja.com/", HEAD);
eval> for (h: response.responseHeader) {print("   " + h.first + ": " + h.second);}
   : HTTP/1.1 200 OK
   Accept-Ranges: bytes
   Arr-Disable-Session-Affinity: true
   Cache-Control: no-cache,public,max-age=1036800
   Content-Length: 45170
   Content-Type: text/html
   Date: Tue, 07 Nov 2017 10:12:53 GMT
   ETag: "4612133abe54d31:0"
   Last-Modified: Fri, 03 Nov 2017 16:10:11 GMT
   Server: Microsoft-IIS/8.0
   Strict-Transport-Security: max-age=31536000
   X-Content-Type-Options: nosniff
   X-Frame-Options: SAMEORIGIN
   X-Powered-By: ASP.NET
eval> var response2=gkScript.httpRequest("http://www.glueckkanja.com/", GET, ["Range": "bytes=0-100"]);
eval> response2.responseBody
<!DOCTYPE html>
<html lang="" class="no-js">

  <head>
    <meta charset="utf-8">
    <meta http-equi
```
