# Errors

``` http
HTTP/1.1 400 OK
Content-Type: application/json

{
    "reason": "Invalid input",
    "errors": [
        {
            "code": 40006,
            "field": "terms.start_date",
            "reason": "start_date is required.", 
            "severity": "Error"
        }
    ]
}
```

### Status Codes

Error Code | Meaning | Description |
---------- | ------- | ------- |
400 | Bad Request | Something is wrong with the request.  Check the error response for details.
401 | Unauthorized | The authorization header is wrong or missing.  Re-authorize at `/oauth/token`.
403 | Forbidden | You're attempting to access a resource that does not belong to you.
404 | Not Found | The resource does not exist.
405 | Method Not Allowed | The resource does not support the method.
500 | Internal Server Error | There's a problem on the DepositGuard side.
503 | Service Unavailable | DepositGuard is temporarially offline for maintanance. Please try again later.

