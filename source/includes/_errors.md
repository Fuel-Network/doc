# Errors

Shipl uses conventional HTTP response codes to indicate the success or failure of an API request. In general: Codes in the 2xx range indicate success. Codes in the 4xx range indicate an error that failed given the information provided (e.g., a required parameter was omitted, a charge failed, etc.). Codes in the 5xx range indicate an error with Shipl's servers (these are rare).

Some 4xx errors that could be handled programmatically include an error code that briefly explains the error reported.


Error Code | Meaning
---------- | -------
200 | OK -- Everything worked as expected.
400 | Bad Request -- The request was unacceptable, often due to missing a required parameter or not JSON
401 | Unauthorized -- No nisaba token
403 | Forbidden -- Nisaba token missing or invalid
404 | Not Found -- The specified path could not be found.
405 | Method Not Allowed -- You tried to access an endpoint with an invalid method.
429 | Too Many Requests -- You're requesting too many transactions! Slow down!
500 | Internal Server Error -- We had a problem with our server. Try again later.
503 | Service Unavailable -- We're temporarily offline for maintenance. Please try again later.