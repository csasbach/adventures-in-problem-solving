# Bell curve polling to prevent hitting rate limits
I was polling a third-party API for blockchain data that would help me compile data for historical price charts for tokens on multiple blockchains.  I wanted the latest data for the charts polled as often as possible but I was hitting rate limits on the API.  So it needed to be very fast, but not too fast. 

Of course, I was making the requests in parallel so they could run as fast as possible, but without any delay, I would get rate-limited instantly.  With too much delay the process would take several minutes, and what was worse is that the request response times varied for each request in ways that were difficult if not impossible to predict, which meant the concurrency of requests would steadily increase until I reached the rate limit again.  If I completely randomized the delay to fall somewhere within the allotted time range, then I'd still have enough 'clumping' of long-running requests that I would experience sporadic rate limiting.

To solve this I wrote logic to evenly distribute all requests on a bell curve.  This can be approximated by averaging multiple random results, and I could 'flatten' the curve by adding 'noise' by doing one more random result and then blending it with the curve result using a weighted average.  I could trim the maximum delay to account for the overall time the process was taking each time it ran to make sure that the added delay would never exceed the time. In this way, I was able to fit the maximum amount of requests into the shortest span of time with the minimum amount of concurrency, thereby avoiding rate limits while keeping the data as fresh as possible.