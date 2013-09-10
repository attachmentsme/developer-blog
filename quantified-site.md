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

This is a story about how we used technologies, such as Nagios, Graphite, StatsD, and Sentry, to gain visibility into our system. Giving us the data we needed to isolate problems, an in turn allowing us to sleep at night.

