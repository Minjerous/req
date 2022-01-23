# req

[![GoDoc](https://pkg.go.dev/badge/github.com/imroc/req.svg)](https://pkg.go.dev/github.com/imroc/req)

A golang http request library for humans.

## Features

* Simple and chainable methods for client and request settings.
* Rich syntax sugar, greatly improving development efficiency.
* Automatically detect charset and decode it to utf-8.
* Powerful debugging capabilities (logging, tracing, and even dump the requests and responses' content).
* All settings can be dynamically adjusted, making it possible to debug in the production environment.
* Easy to integrate with existing code, just replace client's Transport then you can dump content as req to debug APIs.

## Install

``` sh
go get github.com/imroc/req/v2@v2.0.0-alpha.3
```

## Usage

Import req in your code:

```go
import "github.com/imroc/req/v2"
```

### Table of Contents

* [Quick Start](#Quick-Start)
* [Debug](#Debug)
* [PathParam and QueryParam](#PathParam-QueryParam)
* [Header and Cookie](#Header-Cookie)

### <a name="Quick-Start">Quick Start</a>

**Simple GET**

```go
// Create and send a request with the global default client
resp, err := req.New().Get("https://api.github.com/users/imroc")

// Create and send a request with the custom client
client := req.C()
resp, err := client.R().Get("https://api.github.com/users/imroc")
```

**Client Settings**

```go
client.SetUserAgent("my-custom-client").
	SetTimeout(5 * time.Second).
	DevMode()
```

**Request Settings**

```go
var result Result
resp, err := client.R().
	SetResult(&result).
	SetHeader("Accept", "application/json").
	SetQeuryParam("page", "1").
	SetPathParam("userId", "imroc").
	Get(url)
```

### <a name="Debug">Debug</a>

**Dump the content of request and response**

```go
// Set EnableDump to true, default dump all content to stderr,
// including both header and body of request and response
client := req.C().EnableDump(true)
client.R().Get("https://api.github.com/users/imroc")

/* Output
:authority: api.github.com
:method: GET
:path: /users/imroc
:scheme: https
user-agent: custom-client
accept-encoding: gzip

:status: 200
server: GitHub.com
date: Fri, 21 Jan 2022 09:31:43 GMT
content-type: application/json; charset=utf-8
cache-control: public, max-age=60, s-maxage=60
vary: Accept, Accept-Encoding, Accept, X-Requested-With
etag: W/"fe5acddc5c01a01153ebc4068a1f067dadfa7a7dc9a025f44b37b0a0a50e2c55"
last-modified: Thu, 08 Jul 2021 12:11:23 GMT
x-github-media-type: github.v3; format=json
access-control-expose-headers: ETag, Link, Location, Retry-After, X-GitHub-OTP, X-RateLimit-Limit, X-RateLimit-Remaining, X-RateLimit-Used, X-RateLimit-Resource, X-RateLimit-Reset, X-OAuth-Scopes, X-Accepted-OAuth-Scopes, X-Poll-Interval, X-GitHub-Media-Type, X-GitHub-SSO, X-GitHub-Request-Id, Deprecation, Sunset
access-control-allow-origin: *
strict-transport-security: max-age=31536000; includeSubdomains; preload
x-frame-options: deny
x-content-type-options: nosniff
x-xss-protection: 0
referrer-policy: origin-when-cross-origin, strict-origin-when-cross-origin
content-security-policy: default-src 'none'
content-encoding: gzip
x-ratelimit-limit: 60
x-ratelimit-remaining: 59
x-ratelimit-reset: 1642761103
x-ratelimit-resource: core
x-ratelimit-used: 1
accept-ranges: bytes
content-length: 486
x-github-request-id: AF10:6205:BA107D:D614F2:61EA7D7E

{"login":"imroc","id":7448852,"node_id":"MDQ6VXNlcjc0NDg4NTI=","avatar_url":"https://avatars.githubusercontent.com/u/7448852?v=4","gravatar_id":"","url":"https://api.github.com/users/imroc","html_url":"https://github.com/imroc","followers_url":"https://api.github.com/users/imroc/followers","following_url":"https://api.github.com/users/imroc/following{/other_user}","gists_url":"https://api.github.com/users/imroc/gists{/gist_id}","starred_url":"https://api.github.com/users/imroc/starred{/owner}{/repo}","subscriptions_url":"https://api.github.com/users/imroc/subscriptions","organizations_url":"https://api.github.com/users/imroc/orgs","repos_url":"https://api.github.com/users/imroc/repos","events_url":"https://api.github.com/users/imroc/events{/privacy}","received_events_url":"https://api.github.com/users/imroc/received_events","type":"User","site_admin":false,"name":"roc","company":"Tencent","blog":"https://imroc.cc","location":"China","email":null,"hireable":true,"bio":"I'm roc","twitter_username":"imrocchan","public_repos":128,"public_gists":0,"followers":362,"following":151,"created_at":"2014-04-30T10:50:46Z","updated_at":"2021-07-08T12:11:23Z"}
*/
	
// Dump header content asynchronously and save it to file
client := req.C().
	EnableDumpOnlyHeader(). // Only dump the header of request and response
	EnableDumpAsync(). // Dump asynchronously to improve performance
	EnableDumpToFile("reqdump.log") // Dump to file without printing it out
client.Get(url)
```

**Logging**

```go
// Logging is enabled by default, but only output warning and error message to stderr.
// EnableDebug set to true to enable debug level message logging.
client := req.C().EnableDebug(true)
client.R().Get("https://api.github.com/users/imroc")
// Output
// 2022/01/23 14:33:04.755019 DEBUG [req] GET https://api.github.com/users/imroc

// SetLogger with nil to disable all log
client.SetLogger(nil)

// Or customize the logger with your own implementation.
client.SetLogger(logger)
```

**DevMode**

If you want to enable all debug features (dump, debug logging and tracing), just call `DevMode()`:

```go
client := req.C().DevMode()
client.R().Get("https://imroc.cc")
```

### <a name="PathParam-QueryParam">PathParam and QueryParam</a>

```go
client := req.C().DevMode()
client.R().
    SetPathParam("owner", "imroc"). // Set a path param, which will replace the vairable in url path
    SetPathParams(map[string]string{ // Set multiple path params at once 
        "repo": "req",
        "path": "README.md",
    }).SetQueryParam("a", "a"). // Set a query param, which will be encoded as query parameter in url
    SetQueryParams(map[string]string{ // Set multiple query params at once 
        "b": "b",
        "c": "c",
    }).SetQueryString("d=d&e=e"). // Set query params as a raw query string
    Get("https://api.github.com/repos/{owner}/{repo}/contents/{path}?x=x")
/* Output
2022/01/23 14:43:59.114592 DEBUG [req] GET https://api.github.com/repos/imroc/req/contents/README.md?x=x&a=a&b=b&c=c&d=d&e=e
...
*/

// You can also set the common PathParam and QueryParam for every request on client
client.SetPathParam(k1, v1).
    SetPathParams(pathParams).
    SetQueryParam(k2, v2).
    SetQueryParams(queryParams).
    SetQueryString(queryString).
	
resp, err := client.Get(url1)
...

resp, err := client.Get(url2)
...
```

### <a name="Header-Cookie">Header and Cookie</a>

```go
// Let's dump the header to see what's going on
client := req.C().EnableDumpOnlyHeader() 

// Send a request with multiple headers and cookies
resp, err := client.R().
	SetHeader("Accept", "application/json"). // Set one header
    SetHeaders(map[string]string{ // Set multiple headers at once 
        "My-Custom-Header": "My Custom Value",
        "User":             "imroc",
    }).SetCookie(&http.Cookie{ // Set one cookie
        Name:     "imroc/req",
        Value:    "This is my custome cookie value",
        Path:     "/",
        Domain:   "baidu.com",
        MaxAge:   36000,
        HttpOnly: false,
        Secure:   true,
    }).SetCookies([]*http.Cookie{ // Set multiple cookies at once 
        &http.Cookie{
            Name:     "testcookie1",
            Value:    "testcookie1 value",
            Path:     "/",
            Domain:   "baidu.com",
            MaxAge:   36000,
            HttpOnly: false,
            Secure:   true,
        },
        &http.Cookie{
            Name:     "testcookie2",
            Value:    "testcookie2 value",
            Path:     "/",
            Domain:   "baidu.com",
            MaxAge:   36000,
            HttpOnly: false,
            Secure:   true,
        },
    }).Get("https://www.baidu.com/")

/* Output
GET / HTTP/1.1
Host: www.baidu.com
User-Agent: req/v2
Accept: application/json
Cookie: imroc/req="This is my custome cookie value"; testcookie1="testcookie1 value"; testcookie2="testcookie2 value"
My-Custom-Header: My Custom Value
User: imroc
Accept-Encoding: gzip

...
*/

// You can also set the common header and cookie for every request on client.
client.SetHeader(header).
	SetHeaders(headers).
	SetCookie(cookie).
	SetCookies(cookies)

resp, err := client.R().Get(url1)
...
resp, err := client.R().Get(url2)
...
```

## License

Req released under MIT license, refer [LICENSE](LICENSE) file.