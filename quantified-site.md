The Quantified Site
===================

The Attachments.me ecosystem has many dependencies:

* Gmail's IMAP.
* cloud-storage providers: Dropbox, Box, SkyDrive, Google Drive, Egnyte.
* our own [ImageMagick-based thumbnailing service](https://github.com/bcoe/thumbd).
* Crocodoc.

Just to name a few.

A year ago, we realized we had problems. When queues started backing up, and Attachments.me slowed down, it was difficult for us to isolate the exact causes of problems:

* was our database missing an index, resulting in slow queries?
* was our ElasticSearch cluster ready to tip over?

or, was it something outside of our control?

* was Google Drive down?
* did a user have tens-of-thousands of spam-emails pouring into their email account, resulting in a DoS attack on our system?

Our real underlying problem was a lack of visibility into the system. 

This is a story about how we used technologies, such as Nagios, Graphite, StatsD, and Sentry, to gain visibility into our system. This visibility allowed us to rapidly isolate problems in our distributed architecture, reducing stress, and allowing us to iterate faster.

Nagios
------

Nagios monitors key components of our infrastructure, and sends alerts if they're behaving abnornamly; ensuring that my phone rings at 3:00AM if there's a major ops-problem.

Nagios ships with plugins for monitoring various services: _check\_http_, _check\_disk_, _check\_ssh_, etc. Where it really shines, is that it's so easy to extend with your own plugins. We've written plugins for:

* monitoring MongoDB's replication status.
* monitoring the number of concurrent connections to PostgreSQL.
* monitoring the number of messages in our Resque queues.
* monitoring the number of messages in our SQS queues.
* monitoring important metrics in Graphite (more on this later.)

When you release a major bug into produciton, it's useful to have a retrospective. What were the breakdowns in process that allowed for the mistake? Similarly, if we have an operational issue that Nagios fails to notify us of, we attempt to get a check in Nagios going forward.

Here's what we currently have Nagios monitoring:

![Nagios at Attachments.me](./images/quantified-site/nagios.png)

Nagios does a great job of notifying us when a major piece of infrastructe is exploding. It doesn't, however, help us isolate what might be causing the problem. Nor does it tell us when a peice of infrastructure is running, but running in an unexpected manner, e.g., when we've released a buggy version of the website. Sentry, and Graphite, are better suited for these tasks.

Sentry
------

In production, our unhandled exceptions bubble up to [GetSentry](https://getsentry.com/welcome/), a hosted version of David Cramer's [Sentry](https://github.com/getsentry/sentry) library:

* Sentry aggregates together similar exceptions.
* notifies us of newly observed exceptions, as well as regressions.
* and gives us aggregate analytics of exceptions over time.

When releasing Attachments.me, we tend to have a tab open to Sentry's realtime incoming view. If we see a bunch of brand new exceptions flowing in, it's a good indicator that we've released a bug.

Sentry's aggregate metrics can be a good canary-in-the-coal-mine for major issues. For instance, when a cloud storage service recently made a minor unannounced change to their API, it resulted in a huge spike in Sentry exceptions.

![Nagios at Attachments.me](./images/quantified-site/sentry.png)

Graphite/StatsD
---------------

* detailed information about the system, isolate real problems.
* intgegrated with Nagios (this has saved us more than once).

Our Metrics Board
-----------------

* make sure people understand the metrics that are important to ops.
* put them in a central place, let people know when problems are happening.

Conclusion 
----------

* these tools compliment each other.
* a quantified site has made our lives far better.
* I would get this infrastructure in place out-of-the-gate in the future.