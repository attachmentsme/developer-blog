The Quantified Site
===================

The Attachments.me ecosystem has many dependencies:

* Gmail's IMAP.
* cloud-storage providers: Dropbox, Box, SkyDrive, Google Drive, Egnyte.
* our own [ImageMagick-based thumbnailing service](https://github.com/bcoe/thumbd).
* Crocodoc.

Just to name a few...

A year ago, we realized we had problems. When queues started backing up, and Attachments.me slowed down, it was difficult for us to isolate the exact causes of problems:

* was our database missing an index, resulting in slow queries?
* was our ElasticSearch cluster ready to tip over?

or, was it something outside of our control?

* was Google Drive down?
* did a user have tens-of-thousands of spam-emails pouring into their email account, resulting in a DoS attack on our system?

The ultimate problem was a lack of visibility into our system. 

This is a story about how we fixed this problem, using technologies such as: Nagios, Graphite, StatsD, and Sentry. In turn, getting to the point where we had confidence in our system, and could sleep at night.