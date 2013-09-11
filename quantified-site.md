The Quantified Site: How Metrics Drive DevOps at Attachments.me
====================================

Attachments.me has many dependencies:

* Gmail's IMAP.
* cloud-storage providers: Dropbox, Box, SkyDrive, Google Drive, Egnyte.
* our own [image thumbnailing service](https://github.com/bcoe/thumbd).
* Crocodoc.

A year ago, we realized we had problems. When queues started backing up, and Attachments.me slowed down, it was difficult for us to isolate the exact causes of problems:

* was our database missing an index, resulting in slow queries?
* was our ElasticSearch cluster ready to tip over?

or, was it something outside of our control?

* was Google Drive down?
* did a user have tens-of-thousands of spam-emails pouring into their email account, resulting in a DoS attack on our system?

Our real underlying problem was a lack of visibility. 

This is a story about how we used technologies, such as Nagios, Graphite, StatsD, and Sentry, to gain visibility into our system. This visibility allowed us to rapidly isolate problems in our distributed architecture, reducing stress, and allowing us to iterate faster.

Nagios
------

Nagios monitors key components of our infrastructure, and sends alerts if they're behaving abnormally; ensuring that someone's phone rings at 3:00AM if there's a major ops-problem.

Nagios ships with plugins for monitoring various services: _check\_http_, _check\_disk_, _check\_ssh_, etc. Where it really shines, is that it's so easy to extend with your own plugins. We've written plugins for:

* monitoring MongoDB's replication status.
* monitoring the number of concurrent connections to PostgreSQL.
* monitoring the number of messages in our Resque queues.
* monitoring the number of messages in our SQS queues.
* monitoring important metrics in Graphite (more on this later.)

When you release a major bug into production, it's useful to have a retrospective. What were the breakdowns in process that allowed for the mistake? Similarly, if we have an operational issue and Nagios fails to notify us, we put a check in Nagios going forward.

Here's what we currently have Nagios monitoring:

![Nagios at Attachments.me](./images/quantified-site/nagios.png)

Nagios does a great job of notifying us when a major piece of infrastructure is exploding. It doesn't, however, help us isolate what might be causing the problem. Nor does it tell us when a piece of infrastructure is simply running abnormally, e.g., when we've released a buggy version of the website. Sentry, and StatsD/Graphite, are better suited for these tasks.

Sentry
------

In production, our unhandled exceptions bubble up to [GetSentry](https://getsentry.com/welcome/), a hosted version of David Cramer's [Sentry](https://github.com/getsentry/sentry) library:

* Sentry aggregates together similar exceptions.
* notifies us of newly observed exceptions, as well as regressions.
* and gives us aggregate analytics of exceptions over time.

When releasing Attachments.me, we have a browser-tab open to Sentry's realtime incoming view. If we see a bunch of brand new exceptions flowing in, it's a good indicator that we've released a bug:

![Nagios at Attachments.me](./images/quantified-site/incoming.png)

Sentry's aggregate metrics can be a good canary-in-the-coal-mine for major issues. For instance, when a cloud storage service recently made an unannounced change to their API, it resulted in a huge spike in Sentry exceptions:

![Nagios at Attachments.me](./images/quantified-site/sentry.png)

StatsD/Graphite
---------------

StatsD is a daemon that aggregates together statistical-events emitted from your applications. It can, in turn, output this data to Graphite, a tool for visualizing and manipulating graphs.

Attachments.me performs many of operations asynchronously: uploading files to the cloud, thumbnailing images and documents, indexing email. All jobs that we dispatch to a queue are tracked with StatsD:

* an event is emitted when the job is enqueued.
* an event is emitted when the job is pulled off the queue for processing.
* an event is emitted if a job has completed successfully.
* an event is emitted if an exception occurs while processing a job, and it must be retried.
* an event is emitted if a job reaches its maximum retry attempts, and fails.

Emitting StatsD events at these checkpoints, gives us a lot of power to detect anomalies in our system. For instance, if we're enqueuing thousands of Google Drive uploads, but zero are reaching completion, we've got a problem on our hands.

over time, we've determined interesting combinations of graphs that give insight into our system. As an example, here's a dashboard describing the failure rates of different cloud-storage-services:

![Failure Rates of Cloud Services](./images/quantified-site/failure-rates.png)

We've connected Nagios to StatsD, so that we can raise alarms if key-graphs fall into anomalous states. We know, for instance, that attachments-uploaded-per-hour should not fall below the low thousands, unless we're having problems.

![Graphite in Nagios](./images/quantified-site/graphite-nagios.png)

StatsD/Graphite is an invaluable tool for detecting bugs in production. On a few occasions, we've released a major production issue and immediately detected it based on a graph spiking or zeroing-out.

Our Metrics Dashboard
---------------------

As part of our transition towards being a metrics-driven-development-team, we wanted to make sure that key-metrics were always on our mind.

In the center of our office, we now have a television that displays our _Ops Dashboard_:

* it warns us if error rates are on the rise.
* communicates to the rest of the company, if we're currently firefighting an operational meltdown.
* and keeps us appraised of the current state of development: do we have too many open pull-requests, are the builds broken?

![Ops Dashboard](./images/quantified-site/dashboard.png)

Final Thoughts
-------------

The visibility we've gained through Nagios, Sentry, and StatsD/Graphite is incredible. 

* we can submit a fix for a bug, and watch a graph of failure rates drop in response.
* we can break something in production, and with Sentry and Graphite detect the problem, and get it fixed, before a single customer notices.

Getting the point we're at today was an organic process, and an important learning experience for the development team. Having said that, I highly endorse moving towards a similar infrastructure -- it's really quite awesome.

Please tweet at me, or open an issue, if you have any feedback for this post :thumbsup:

-----------------
_Benjamin Coe_ is the co-founder of [Attachments.me](https://attachments.me), he can often be found [tweeting](https://twitter.com/#/benjamincoe) and [coding](https://github.com/bcoe).
