# Script-based ticket system integration
A new ticketing system for both internal and external business partners was due by the end of the quarter and corporate had promised the external partners this part of the integration would be testable before that deadline.  The complication was integration with that new ticketing system was tied up in a much larger project to completely replace our existing monitoring infrastructure with a new one in the cloud.  That project was not meeting targets for completion and definitely going to go over this particular red line.

So the question was raised, can we, in two weeks, build an integration with the legacy monitoring system that would work with both the old and the new ticketing systems, that would not be a part of or depend on the larger project?  In that meeting, I and the experts in the systems to be integrated proposed the following solution:  

I would write a few Powershell scripts that would run on the server hosting the legacy monitoring software that would be run on a cron schedule.  It was the job of those scripts to pick up any messages that have not yet been processed and forward them to the appropriate ticketing system (based on a value in configuration) and then verify those messages were received by the destination system.  

Because these were scripts and not a properly deployed and installed application, there were risks that needed to be mitigated but this was the design we could complete in two weeks.  I mitigated those risks in these ways:

We had other monitoring systems that we relied upon to tell us when the main monitoring systems were down.  I used these to send an alert anytime the scripts stopped sending their keep-alive signals.  We could also use these to alert on misconfiguration.  Because the scripts ran on a cron we didn't need to retry events, we just needed logic that made sure every event was eventually processed in an idempotent way.  If the same event(s) failed enough times in a row to exceed a threshold then an alert was sent.

Not only did the telemetry built into the scripts do the job of making this stopgap solution reliable but in the week of testing after it was completed many issues were found and resolved that were already present in the legacy integration that were only found because of the new logging.

It's been more than a couple of years since I left that company but my former colleagues have informed me that they are still using that solution and that it is very easy for them to maintain because, as operations people, they don't typically write application code but they are comfortable with Powershell and therefore the alerts are actionable for them.
