=================================================================
How to setup Cross-Origin Resource Sharing (CORS) on DreamObjects
=================================================================

Overview
~~~~~~~~

This article describes how to set up the Cross-Origin Resource Sharing
(**CORS**) capabilities of DreamObjects as implemented in Ceph, and is
intended for any users that need to set up DreamObjects for use across domains,
such as WebFonts, or cross-domain uploads.

Background and use cases
~~~~~~~~~~~~~~~~~~~~~~~~

Cross-origin resource sharing (CORS) is a mechanism that allows
restricted resources (e.g., fonts) on a web page to become available to another
domain outside the domain from which the resource originated. A restricted
resource is any that would violate the same-origin policy of the browser.

In plain terms, a CORS policy request is how the browser determines if it’s
actually permitted to perform non-trivial requests from one domain to another.
Some canonical examples of these are WebFonts and JavaScript-based
(XMLHttpRequest) uploads to an external storage service.

Use case: Cross-domain uploads
------------------------------

Historically, if a site had file upload functionality, the upload (via a form)
could occur to any site. The browser did not consider any security implications
of the POST to another site. JavaScript code however was subject to the
same-origin policy, and not permitted to contact another domain. Fancy upload
forms using JavaScript and XMLHttpRequest were greatly limited by the
same-origin policy, especially as cloud storage services increased in
popularity. A CORS policy for uploading files using XMLHttpRequest must specify
that the browser is permitted to POST/PUT a file if it’s coming from a given
website.

Use case: WebFont usage
-----------------------

WebFonts use the same-origin policy as part of an answer to the DRM needs to
prevent fonts from being used outside of their licensing terms. Leaving aside
any discussion of the effectiveness of this policy, web fonts must be able to
work on web sites with specific graphic design needs. The CORS policy for a
WebFont must specify that it’s permitted to download (GET request) the WebFont
for use on a given website.

This same-origin policy is mandated by the `CSS3-Fonts specification <http://www.w3.org/TR/css3-fonts/#same-origin-restriction>`_.

DreamObjects CORS usage
~~~~~~~~~~~~~~~~~~~~~~~

DreamObjects provides CORS-policy responses to browser requests, based on the
CORS configuration.

A CORS configuration on DreamObjects:

* includes what site a request is for as well as what type of request,
* is handled individually for each bucket, and
* uses the Amazon S3 syntax for CORS configuration.

Constructing a CORS configuration
---------------------------------

Rules for CORS policies
^^^^^^^^^^^^^^^^^^^^^^^

The following are the general rules for making a CORS configuration:

* A valid CORS configuration consists of 0 to 100 CORS rules.
* Each rule must include at least one origin.
* An origin may contain at most one wildcard **\***
* Each rule must include at least one method.
* The supported methods are: GET, HEAD, PUT, POST, DELETE.
* Each rule may contain an identifying string of up to 255 characters.
* Each rule may specify zero or more allowed request headers (which the client
  may include in the request).
* Each rule may specify zero or more exposed response headers (which are sent
  back from the server to the client).
* Each rule may specify a cache validity time of zero or more seconds. If not
  included, the client should supply their own default.

Example WebFont policy
^^^^^^^^^^^^^^^^^^^^^^

If you need to host a WebFont on DreamObjects, you’ll want to include a
policy such as the following example (assuming your site is
www.example.com and also works at example.com):

.. code::

    <CORSConfiguration>
    <CORSRule>
        <ID>Allow WebFont for example.com</ID>
        <AllowedOrigin>https://www.example.com</AllowedOrigin>
        <AllowedOrigin>http://www.example.com</AllowedOrigin>
        <AllowedOrigin>https://example.com</AllowedOrigin>
        <AllowedOrigin>http://example.com</AllowedOrigin>
        <AllowedMethod>GET</AllowedMethod>
        <AllowedMethod>HEAD</AllowedMethod>
        <AllowedHeader>Content-*</AllowedHeader>
        <AllowedHeader>Host</AllowedHeader>
        <ExposeHeader>ETag</ExposeHeader>
        <MaxAgeSeconds>86400</MaxAgeSeconds>
    </CORSRule>
    </CORSConfiguration>

Example AWS S3 JS policy
^^^^^^^^^^^^^^^^^^^^^^^^

The following policy permits users of the AWS S3
JavaScript SDK, on both example.com and
www.example.com, on both HTTP and HTTPS, to upload to
DreamObjects, with both the PUT and POST methods:

.. code::

    <CORSConfiguration>
    <CORSRule>
        <ID>example.com: Allow PUT & POST with AWS S3 JS
        SDK</ID>
        <AllowedOrigin>https://www.example.com</AllowedOrigin>
        <AllowedOrigin>http://www.example.com</AllowedOrigin>
        <AllowedOrigin>https://example.com</AllowedOrigin>
        <AllowedOrigin>http://example.com</AllowedOrigin>
        <AllowedMethod>PUT</AllowedMethod>
        <AllowedMethod>POST</AllowedMethod>
        <AllowedHeader>Origin</AllowedHeader>
        <AllowedHeader>Content-Length</AllowedHeader>
        <AllowedHeader>Content-Type</AllowedHeader>
        <AllowedHeader>Content-MD5</AllowedHeader>
        <AllowedHeader>X-Amz-User-Agent</AllowedHeader>
        <AllowedHeader>X-Amz-Date</AllowedHeader>
        <AllowedHeader>Authorization</AllowedHeader>
        <ExposeHeader>ETag</ExposeHeader>
        <MaxAgeSeconds>1800</MaxAgeSeconds>
    </CORSRule>
    <CORSRule>
        <ID>example.com: Allow GET with AWS S3 JS SDK</ID>
        <AllowedOrigin>*</AllowedOrigin>
        <AllowedMethod>GET</AllowedMethod>
        <AllowedMethod>HEAD</AllowedMethod>
        <AllowedHeader>*</AllowedHeader>
        <ExposeHeader>ETag</ExposeHeader>
        <MaxAgeSeconds>1800</MaxAgeSeconds>
    </CORSRule>
    </CORSConfiguration>

Example Wildcard policy (*INSECURE!*)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The following policy, while completely insecure, allows ALL methods from any
origin.  It does NOT however expose custom headers:

.. code::

    <CORSConfiguration>
    <CORSRule>
        <ID>Allow
        everything</ID>
        <AllowedOrigin>*</AllowedOrigin>
        <AllowedMethod>GET</AllowedMethod>
        <AllowedMethod>HEAD</AllowedMethod>
        <AllowedMethod>PUT</AllowedMethod>
        <AllowedMethod>POST</AllowedMethod>
        <AllowedMethod>DELETE</AllowedMethod>
        <AllowedHeader>*</AllowedHeader>
        <MaxAgeSeconds>30</MaxAgeSeconds>
    </CORSRule>
    </CORSConfiguration>

Deploying a CORS configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
A minority of S3 clients support deploying CORS configurations.  Some (such as
boto) also support programmatically constructing a CORS configuration.  (See
the links in the **clients** section below for examples of deploying
a CORS configuration on various clients.) Other clients not listed may also
support CORS policies, and the listing should not be taken as exhaustive or
guaranteed correct (some clients have experienced broken CORS support at some
points).

s3cmd (1.6.0 and newer)
-----------------------

Since 1.6.0, s3cmd supports setting or deleting a CORS config; however it does
not support getting it back except as a part of an "info" request.

.. note:: Make sure you have a working `installation of S3cmd
          <215916627>`_ before proceeding.

.. code-block:: bash

    # Set the CORS rules
    s3cmd setcors rules.xml s3://bucketname
    # Delete the CORS rules
    s3cmd delcors s3://bucketname
    # Get bucket info including CORS rules
    s3cmd info s3://bucketname

Python/BOTO (pre-made XML)
--------------------------

The following is a minimal snippet of boto Python to deploy a CORS
configuration to DreamObjects:

.. code-block:: python

    from boto.s3.connection import S3Connection
    host = 'objects-us-west-1.dream.io'
    access_key = '...'
    secret_key = '...'
    conn = S3Connection(
        host=host,
        aws_access_key_id=access_key,
        aws_secret_access_key=secret_key,
        )
    my_cors_conf = """
    <CORSConfiguration>
    <!-- policy goes here -->
    </CORSConfiguration>
    """
    bucket = conn.get_bucket('examplebucketname')
    bucket.set_cors_xml(my_cors_conf)

Python/BOTO (Programmatic)
--------------------------

The following is a minimal snippet of boto Python to construct and deploy CORS
configuration to DreamObjects:

.. code-block:: python

    import boto.s3.connection
    import boto.s3.cors
    import itertools

    host = 'objects-us-west-1.dream.io'
    access_key = '...'
    secret_key = '...'
    bucket_name = '...'

    conn = boto.s3.connection.S3Connection(host=host, aws_access_key_id=access_key, aws_secret_access_key=secret_key)
    bucket = conn.get_bucket(bucket_name)

    try:
        corsobj = bucket.get_cors()
    except:
        corsobj = boto.s3.cors.CORSConfiguration()

    id = 'DH-CORS-Example-ID1234' # each rule MAY have an optional ID, and if so they MUST be unique
    domains = ['example.com', 'demo.com', '...' ] # edit as needed
    methods = ['GET', 'HEAD', 'PUT', 'POST', 'DELETE' ] # edit as needed, this covers AWS JS SDK + WebFont
    ahdr = ['Authorization', 'Content-*', 'X-Amz-*', 'Origin', 'Host'] # edit as needed, this covers AWS JS SDK + WebFont
    ehdr = ['ETag', 'Content-MD5']

    # Construct the origins from domains, allowing HTTP, HTTPS, on the domain with and without 'www.'
    # If you want to require HTTPS, you should remove 'http' element from the first list
    protocols = ['http', 'https']
    domain_prefix = ['','www.']
    origins_tuple = itertools.product(protocols, domain_prefix, domains)
    origins = ['{0}://{1}{2}'.format(*t) for t in origins_tuple]

    # Add the rule to the CORS object
    corsobj.add_rule(
        methods,
        origins,
        id=id,
        allowed_header=ahdr,
        max_age_seconds=3600,
        expose_header=ehdr,
        )

    # This little bit of magic allows us to deduplicate CORS rules:
    # 1. Allow us to compare CORSRule elements
    def CORSRule_eq(self, other):
        return self.__dict__ == other.__dict__

    boto.s3.cors.CORSRule.__eq__ = CORSRule_eq
    # 2. Now find unique elements
    corsobj = boto.s3.cors.CORSConfiguration([key for key,_ in itertools.groupby(corsobj)])

    # Put the updated CORS on the bucket
    bucket.set_cors(corsobj)

Compatibility notes
~~~~~~~~~~~~~~~~~~~

* DreamObjects was originally implemented with only a default CORS policy of
  the **\*** wildcard, which permitted ANY origin to be used; no
  per-bucket CORS was originally available.
* As of 2015/10/01, per-bucket CORS policies are fully supported, but the
  wildcard in some places remained in place to avoid inadvertent breakages.
* As of 2016/02/01, this wildcard became unavailable, and users who need
  CORS functionality MUST deploy their own CORS configuration to the relevant
  buckets.

See also
~~~~~~~~

Background
----------

* `Cross-site xmlhttprequest with CORS <https://hacks.mozilla.org/2009/07/cross-site-xmlhttprequest-with-cors/>`_
* `Wikipedia: CORS <https://en.wikipedia.org/wiki/Cross-origin_resource_sharing>`_
* `W3 CORS specification <http://www.w3.org/TR/cors/>`_
* `Wikipedia: Same-origin policy <https://en.wikipedia.org/wiki/Cross-origin_resource_sharing>`_

Clients
-------

* `S3 Browser: Bucket CORS Configuration <http://s3browser.com/s3-bucket-cors-configuration.php>`_
* `boto: S3 <https://boto.readthedocs.io/en/latest/ref/s3.html>`_
* `Bucket Explorer: Amazon S3 - Manage Cross-Origin Resource Sharing (CORS) <http://www.bucketexplorer.com/documentation/amazon-s3--manage-cross-origin-resource-sharing.html>`_
* `CyberDuck: Not supported as of 2015/09/29 <https://trac.cyberduck.io/wiki/help/en/howto/s3>`_

API
---

* `Working with Amazon S3 Objects &lt;&lt; Enabling Cross-Origin Resource Sharing <http://docs.aws.amazon.com/AmazonS3/latest/dev/cors.html>`_
* `Amazon S3: REST API, Bucket PUT CORS <http://docs.aws.amazon.com/AmazonS3/latest/API/RESTBucketPUTcors.html>`_
* `Amazon S3: REST API, Bucket DELETE CORS <http://docs.aws.amazon.com/AmazonS3/latest/API/RESTBucketDELETEcors.html>`_
* `Amazon S3: REST API, Bucket GET CORS <http://docs.aws.amazon.com/AmazonS3/latest/API/RESTBucketGETcors.html>`_

.. meta::
    :labels: CORS
