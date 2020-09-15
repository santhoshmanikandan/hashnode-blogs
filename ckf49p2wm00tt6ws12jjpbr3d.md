## Tune your Apache Tomcat servers for your load

With tomcat 8.xx and above, tomcat uses NIO2/APR as a connector by default which foregoes one thread for each incoming request model(BIO). It uses a common thread pool for accepting requests which then forwards the requests for processing threads if available. Also when the request is handed over to processing threads, the acceptor thread is free to process newer requests.

The key which configures this value is maxConnections. From  [Tomcat 8.5 Reference](https://tomcat.apache.org/tomcat-8.5-doc/config/http.html),


> The maximum number of connections that the server will accept and process at any given time. When this number has been reached, the server will accept, but not process, one further connection. This additional connection be blocked until the number of connections being processed falls below maxConnections at which point the server will start accepting and processing new connections again. Note that once the limit has been reached, the operating system may still accept connections based on the acceptCount setting. The default value varies by connector type. For NIO and NIO2 the default is 10000. For APR/native, the default is 8192.

Though this looks good on paper, these changes can be counterproductive for application which are time-intensive by nature. Take for an example, a tomcat server application which parses a large file or process which indexes a file on request and consider each process takes on an average 30-60s. So to handle this load assume maxThreads is configured as 20.

A sudden spike in incoming requests can fill the acceptor queue and the server is now unavailable to accept new requests for ((10000/20) * 60 = 30000 seconds or 3.1 hours!!!!). Things become worse if the client read times out after as the server will be processing requests which is no longer required. The easiest way to handle this situation is to restart the tomcat process if the port fails to reply to a health check request for some configured time(5 min). But this will result in loss of all the request data which reached the server and also will result in unwanted downtime of the service.

Therefore, tuning the tomcat config is required for efficient use of servers. Always start with a low value and increase it so that you can see the trend in the increase in resource used, time taken for processing etc. 

In the above example, you can start with maxConnections same as that of maxThreads and increase if the client doesn't read time out often.

Another parameter which handles incoming queue is acceptQueue,

>The maximum queue length for incoming connection requests when all possible request processing threads are in use. Any requests received when the queue is full will be refused. The default value is 100.

Similar to maxConnections, acceptQueue requests too get processed irrespective of client alive status. Start with less value like 10 or so and increase it with the requirement.

But remember this, increasing the maxConnections and acceptQueue to accept all the incoming requests will do more harm than good as the client could close the connection by the time the request in the queue get's processed. 

Once incoming queue gets full, connectTimeOut error will occur in client end and if there are more number of such errors, it is an indication to scale the server horizontally.