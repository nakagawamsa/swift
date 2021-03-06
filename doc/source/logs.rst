====
Logs
====

Swift has quite verbose logging, and the generated logs can be used for
cluster monitoring, utilization calculations, audit records, and more. As an
overview, Swift's logs are sent to syslog and organized by log level and
syslog facility. All log lines related to the same request have the same
transaction id. This page documents the log formats used in the system.

----------
Proxy Logs
----------

The proxy logs contain the record of all external API requests made to the
proxy server. Swift's proxy servers log requests using a custom format
designed to provide robust information and simple processing. The log format
is::

    client_ip remote_addr datetime request_method request_path protocol
        status_int referer user_agent auth_token bytes_recvd bytes_sent
        client_etag transaction_id headers request_time source log_info
        request_start_time request_end_time

=================== ==========================================================
**Log Field**       **Value**
------------------- ----------------------------------------------------------
client_ip           Swift's guess at the end-client IP, taken from various
                    headers in the request.
remote_addr         The IP address of the other end of the TCP connection.
datetime            Timestamp of the request, in
                    day/month/year/hour/minute/second format.
request_method      The HTTP verb in the request.
request_path        The path portion of the request.
protocol            The transport protocol used (currently one of http or
                    https).
status_int          The response code for the request.
referer             The value of the HTTP Referer header.
user_agent          The value of the HTTP User-Agent header.
auth_token          The value of the auth token. This may be truncated or
                    otherwise obscured.
bytes_recvd         The number of bytes read from the client for this request.
bytes_sent          The number of bytes sent to the client in the body of the
                    response. This is how many bytes were yielded to the WSGI
                    server.
client_etag         The etag header value given by the client.
transaction_id      The transaction id of the request.
headers             The headers given in the request.
request_time        The duration of the request.
source              The "source" of the reuqest. This may be set for requests
                    that are generated in order to fulfill client requests,
                    e.g. bulk uploads.
log_info            Various info that may be useful for diagnostics, e.g. the
                    value of any x-delete-at header.
request_start_time  High-resolution timestamp from the start of the request.
request_end_time    High-resolution timestamp from the end of the request.
=================== ==========================================================

In one log line, all of the above fields are space-separated and url-encoded.
If any value is empty, it will be logged as a "-". This allows for simple
parsing by splitting each line on whitespace. New values may be placed at the
end of the log line from time to time, but the order of the existing values
will not change. Swift log processing utilities should look for the first N
fields they require (e.g. in Python using something like
``log_line.split()[:14]`` to get up through the transaction id).


-----------------
Storage Node Logs
-----------------

Swift's account, container, and object server processes each log requests
that they receive. The format for these log lines is::

    remote_addr - - [datetime] "request_method request_path" status_int
        content_length "referer" "transaction_id" "user_agent" request_time

=================== ==========================================================
**Log Field**       **Value**
------------------- ----------------------------------------------------------
remote_addr         The IP address of the other end of the TCP connection.
datetime            Timestamp of the request, in
                    "day/month/year:hour:minute:second +0000" format.
request_method      The HTTP verb in the request.
request_path        The path portion of the request.
status_int          The response code for the request.
content_length      The value of the Content-Length header in the response.
referer             The value of the HTTP Referer header.
transaction_id      The transaction id of the request.
user_agent          The value of the HTTP User-Agent header. Swift's proxy
                    server sets its user-agent to
                    ``"proxy-server <pid of the proxy>".``
request_time        The duration of the request.
=================== ==========================================================
