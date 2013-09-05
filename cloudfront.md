Fun and Profit, Through Amazon CloudFront
=====================================

_Can we speed up the load time of landing page to sub-2-seconds?_, this ticket had been kicking around in our backlog for quite some time. We'd consistently made attempts to deliver assets efficiently:

* CSS and JavaScript was combined and minified.
* static assets were served directly through Nginx.
* Nginx was configured to gzip the assets that it delivered.

Regardless of these efforts, our landing page's loading times were becoming abysmal:

![Before CloudFront](./images/cloudfront/before-cf.png)

As the graph above shows, our load-times were pushing 8-seconds!

AWS provides a Content Distribution Network called CloudFront. I had wanted to try out CloudFront for quite some time, figuring that it would be a quick performance win; While I expected speed improvements, I was blown away by just how much faster load times ended up being.

In this post I'll:

* give a quick introduction to setting up CloudFront.
* outline how we moved our assets over to CloudFront in an incremental manner.
* briefly go over some of the issues we encountered.
* discuss the performance improvements we ultimately witnessed.

How CloudFront Works
--------------------

To start using CloudFront, you create what is called a Distribution. A Distribution consists of an Origin Domain Name, and various configuration settings (mostly related to how assets are cached). When a visitor to your website requests an asset from the CDN, if the asset has already been retrieved it will be served immediately. If a user requests an asset that has never been pulled into the CDN, the CDN will attempt to load the asset from the Origin Domain Name. Let's look at a real-world example:

* Attachments.me has a CloudFront Distribution at the URL https://drvlo06w0956l.cloudfront.net, with the Orgin Domain Name of https://attachments.me
* Let's suppose that a user requests the asset: https://drvlo06w0956l.cloudfront.net/images/marketing/logo.png
* If logo.png is already cached in CloudFront, it will be served immediately.
* If logo.png is not already cached in CloudFront, the asset will be loaded from the Origin Domain Name: https://attachments.me/images/marketing/logo.png

So simple! CloudFront accepts a request for an asset at a given path, either servers the asset, or loads the asset from the corresponding path of the Origin Domain Name.

Moving Assets over to CloudFront
--------------------------------

Rather than immediately switching our entire site to using CloudFront, we decided to incrementally move a few key pages over. This allowed us meassure performance, experiment with cache settings, and to perform various other sanity checks, before fully-committing to CloudFront. We created several [Rails View Helpers](https://github.com/attach
mentsme/cloud_front_helpers) to aid us in this incremental approach.



Some Caveats
------------

The Final Numbers
-----------------


