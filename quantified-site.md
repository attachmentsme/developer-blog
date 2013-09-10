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

Nagios monitors key components of our infrastructure, and sends alerts if they're behaving abnornamly; ensuring that my phone rings at 3:00AM in the morning, if there's a major ops-problem.

Nagios ships with plugins for monitoring various services: _check\_http_, _check\_disk_, _check\_ssh_, etc. Where it really shines, is that it's so easy to extend with your own plugins. We've written plugins for:

* monitoring MongoDB's replication status.
* monitoring the number of concurrent connections to PostgreSQL.
* monitoring the number of messages in our Resque queues.
* monitoring the number of messages in our SQS queues.
* monitoring important metrics in Graphite (more on this later.)

When you release a major bug into produciton, it's useful to have a retrospective. What breakdowns in process that allowed for the mistake? Similarly, if we have an operational issue that Nagios fails to notify us of, we attempt to get a check in Nagios going forward.

Here's what we currently have Nagios monitoring:

![Nagios at Attachments.me](./images/quantified-site/nagios.png)

Sentry
------

* great way to isolate regressions, find bugs in the code we release.
* canary in the coal-mine, are graphs significantly higher than they used to be.

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