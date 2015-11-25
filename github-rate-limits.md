API rate limits
----------------
Use case – [API access rate limiting mechanism](https://developer.github.com/v3/#rate-limiting) at [GitHub](https://github.com)

GitHub implements a very easy to understand and easy to use rate limiting system.

* By default, anybody can reach GitHub's API end points, authenticated or otherwise
* GitHub limits all unauthenticated requests originating from an IP address (typically to 60 requests per hour)
* Every response sent out by GitHub APIs will have three elements in response headers, namely – `X-RateLimit-Limit`, `X-RateLimit-Remaining` and `X-RateLimit-Reset`
    * `X-RateLimit-Limit` – what is the total limit for unit period in time, which in Github's case is 60 minutes
    * `X-RateLimit-Reset` - when would GitHub reset the limit, represented as UNIX timestamp of seconds since epoch
    * `X-RateLimit-Remaining` - how many requests can one fire till the limit reset time
* GitHub limits all authenticated requests from an authenticated user (typically to 5000 requests per hour)

Example. An unauthenticated request to GitHub API end point gives the response header as shown below. In the below given example, I am trying to reach
`https://api.github.com/users/rghv/repos`, which is the end-point for getting list of the repositories of a user named `rghv`.

```
vinay@vinay-lpt-lnx ~ $ date
Tue Nov 24 17:20:50 IST 2015
vinay@vinay-lpt-lnx ~ $ curl -I https://api.github.com/users/rghv/repos
HTTP/1.1 200 OK
Server: GitHub.com
Date: Tue, 24 Nov 2015 11:51:05 GMT
Content-Type: application/json; charset=utf-8
Content-Length: 149096
Status: 200 OK
X-RateLimit-Limit: 60
X-RateLimit-Remaining: 45
X-RateLimit-Reset: 1448368114
Cache-Control: public, max-age=60, s-maxage=60
ETag: "cc69d58be36b2f30ae55bf315990dd69"
Vary: Accept
X-GitHub-Media-Type: github.v3
X-XSS-Protection: 1; mode=block
X-Frame-Options: deny
Content-Security-Policy: default-src 'none'
Access-Control-Allow-Credentials: true
Access-Control-Expose-Headers: ETag, Link, X-GitHub-OTP, X-RateLimit-Limit, X-RateLimit-Remaining, X-RateLimit-Reset, X-OAuth-Scopes, X-Accepted-OAuth-Scopes, X-Poll-Interval
Access-Control-Allow-Origin: *
Strict-Transport-Security: max-age=31536000; includeSubdomains; preload
X-Content-Type-Options: nosniff
Vary: Accept-Encoding
X-Served-By: 2d7a5e35115884240089368322196939
X-GitHub-Request-Id: CBC93E72:7FBA:759C249:56544F27

vinay@vinay-lpt-lnx ~ $ date --date="@1448368114"
Tue Nov 24 17:58:34 IST 2015
vinay@vinay-lpt-lnx ~ $
```
The results shown above is what is called 'response headers'. The response headers contain very valuable information.
Notice the `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset` values. The request was fired on Nov 24 17:20:50 IST 2015.
`X-RateLimit-Limit` mentions that the hourly limit is 60, out of which 45 have been used (as shown by `X-RateLimit-Remaining`). The limit
would reset to 60 at Nov 24 17:58:34 IST 2015 as mentioned by `X-RateLimit-Reset`, which is depicted in seconds since UNIX epoch.

Now, when the same end-point is reached with an authenticated request, the same response headers are seen but with better limits, since the
requests are verified to be coming from a valid authenticated user.

```
vinay@vinay-lpt-lnx ~ $ curl -I -H "Authorization: token superawesomesampleapitoken" https://api.github.com/users/rghv/repos
HTTP/1.1 200 OK
Server: GitHub.com
Date: Tue, 24 Nov 2015 11:35:27 GMT
Content-Type: application/json; charset=utf-8
Content-Length: 151736
Status: 200 OK
X-RateLimit-Limit: 5000
X-RateLimit-Remaining: 4997
X-RateLimit-Reset: 1448368360
Cache-Control: private, max-age=60, s-maxage=60
ETag: "f2ee1838419c60cb3d30def6489aef45"
X-OAuth-Scopes: gist, repo, user
X-Accepted-OAuth-Scopes: 
Vary: Accept, Authorization, Cookie, X-GitHub-OTP
X-GitHub-Media-Type: github.v3
X-XSS-Protection: 1; mode=block
X-Frame-Options: deny
Content-Security-Policy: default-src 'none'
Access-Control-Allow-Credentials: true
Access-Control-Expose-Headers: ETag, Link, X-GitHub-OTP, X-RateLimit-Limit, X-RateLimit-Remaining, X-RateLimit-Reset, X-OAuth-Scopes, X-Accepted-OAuth-Scopes, X-Poll-Interval
Access-Control-Allow-Origin: *
Strict-Transport-Security: max-age=31536000; includeSubdomains; preload
X-Content-Type-Options: nosniff
Vary: Accept-Encoding
X-Served-By: 13d09b732ebe76f892093130dc088652
X-GitHub-Request-Id: CBC93E72:94AE:2B2A0B78:56544B7E

vinay@vinay-lpt-lnx ~ $ 
```

