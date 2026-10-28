502 vs 503 vs 504 — Is the Load Balancer Really Down?
Many engineers see 502, 503, or 504 and immediately think:
“The load balancer is down.”
But that’s not always the case.
In fact, if the load balancer process is completely stopped or the host has lost network connectivity, the client usually won’t receive an HTTP 5xx response at all.
Instead, you may see:
ERR_CONNECTION_REFUSED
TCP connection timeout
Connection reset
“Unable to connect”
Why?
Because there is no process available to generate an HTTP response.
So when do we get 502, 503, and 504?
The load balancer or reverse proxy is usually still running, but it is having trouble communicating with the upstream service.
A simple way to remember them:
🔴 502 Bad Gateway → The upstream responded, but the response was invalid or unexpected.
🟠 503 Service Unavailable → No healthy backend is currently available to handle the request.
🟡 504 Gateway Timeout → The proxy connected or attempted to connect to the upstream, but the upstream didn't respond within the configured timeout.
My simple mental model
502 = Bad response
503 = No healthy backend
504 = Too slow
Think about the request path
Client    ↓ Load Balancer / Nginx    ↓ Frontend / Backend    ↓ Database
If the load balancer is running but cannot successfully communicate with the backend, it may generate a 502, 503, or 504.
If the backend application itself throws an error, you may see a 500 Internal Server Error.
And if the database is unavailable, the application might translate that failure into a 500 or 503, depending on how the application handles the error.
What I check during an incident
When I see a 5xx error, I don't immediately restart the load balancer.
I usually check layer by layer:
1️⃣ Is the load balancer actually responding?
2️⃣ What do the load balancer/reverse-proxy logs show?
3️⃣ Are the upstream targets healthy?
4️⃣ Can the load balancer connect to the backend?
5️⃣ Is the backend responding within the expected timeout?
6️⃣ Are there any application or database errors?
The key takeaway
A 502/503/504 can actually be evidence that the load balancer is alive enough to return an HTTP response.
If the load balancer is completely gone, you may not get an HTTP status code at all — you may get a TCP/network-level connection failure instead.
Understanding this small distinction can make troubleshooting much faster.
Don't just memorize 502, 503, and 504. Understand where the failure is happening in the request path.