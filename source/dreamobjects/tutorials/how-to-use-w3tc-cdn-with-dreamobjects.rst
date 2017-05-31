===============================================
How to Use W3 Total Cache CDN with DreamObjects
===============================================

Is it possible to connect my DreamObjects bucket to the W3 Total Cache
WordPress plugin's CDN? The plugin asks for a CDN type and
DreamObjects isn't in the list but I'm wondering if any other of those
choices will work:

https://cl.ly/260R2122070F

Yes, it is possible. From that list, select "Origin Push: Amazon
Simple Storage Service (S3) Compatible" as the CDN Type. Then, on the
[Performance -> CDN -> Configuration] page, use the following values.

- API host: `objects-us-west-1.dream.io`
- Access key ID / Secret key: These can be located on the [Cloud
Services -> DreamObjects -> Manage] page of the panel
- Bucket: Specify whatever bucket you prefer (although for the SSL
option, buckets without dots/periods `.` will work best)
- SSL support: Per above, if your bucket name does not contain a `.` in
it, you can use the "Auto (determine connection type automatically)"
option.
- Replace site's hostname with: You'll need to setup a custom CNAME
alias for your bucket in the panel and then enter that here (we also
recommend that you enable DreamSpeed CDN to avoid lag between your VPS
in our Ashburn, Virginia, USA data center and the DreamObjects service
in our Irvine, California, USA data center).

Once you've done all that, the "Test S3 upload" button should report
"Test passed" and you can start using the "Upload..." buttons in the
"General" section of the [Performance -> CDN] page of the WordPress
dashboard to upload the necessary WordPress content to your
DreamObjects bucket.

.. meta::
    :labels: cdn
