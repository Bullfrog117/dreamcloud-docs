Sample Ruby Code Using DreamObjects S3-compatible API
=====================================================

.. container:: table_of_content

    - :ref:`S3_Ruby_Listing_Owned_Buckets`
    - :ref:`S3_Ruby_Creating_A_Bucket`
    - :ref:`S3_Ruby_Listing_A_Buckets_Content`
    - :ref:`S3_Ruby_Deleting_A_Bucket`
    - :ref:`S3_Ruby_Forced_Delete_For_Non-Empty_Buckets`
    - :ref:`S3_Ruby_Creating_An_Object`
    - :ref:`S3_Ruby_Change_An_Objects_ACL`
    - :ref:`S3_Ruby_Download_An_Object`
    - :ref:`S3_Ruby_Delete_An_Object`
    - :ref:`S3_Ruby_Generate_Object_Download_URLs`

.. _S3_Ruby_Creating_A_Connection:

Creating a Connection
---------------------

This creates a connection so that you can interact with the server.

.. code-block:: ruby

    AWS::S3::Base.establish_connection!(
            :server            => 'objects-us-west-1.dream.io',
            :use_ssl           => true,
            :access_key_id     => 'my-access-key',
            :secret_access_key => 'my-secret-key'
    )


.. _S3_Ruby_Listing_Owned_Buckets:

Listing Owned Buckets
---------------------

This gets a list of `AWS::S3::Bucket`_ objects that you own.
This also prints out the bucket name and creation date of each bucket.

.. code-block:: ruby

    AWS::S3::Service.buckets.each do |bucket|
            puts "#{bucket.name}\t#{bucket.creation_date}"
    end

The output will look something like this::

   mahbuckat1	2011-04-21T18:05:39.000Z
   mahbuckat2	2011-04-21T18:05:48.000Z
   mahbuckat3	2011-04-21T18:07:18.000Z


.. _S3_Ruby_Creating_A_Bucket:

Creating a Bucket
-----------------

This creates a new bucket called ``my-new-bucket``

.. code-block:: ruby

    AWS::S3::Bucket.create('my-new-bucket')


.. _S3_Ruby_Listing_A_Buckets_Content:

Listing a Bucket's Content
--------------------------

This gets a list of hashes with the contents of each object
This also prints out each object's name, the file size, and last
modified date.

.. code-block:: ruby

    new_bucket = AWS::S3::Bucket.find('my-new-bucket')
    new_bucket.each do |object|
            puts "#{object.key}\t#{object.about['content-length']}\t#{object.about['last-modified']}"
    end

The output will look something like this if the bucket has some files::

   myphoto1.jpg	251262	2011-08-08T21:35:48.000Z
   myphoto2.jpg	262518	2011-08-08T21:38:01.000Z


.. _S3_Ruby_Deleting_A_Bucket:

Deleting a Bucket
-----------------
.. note::
   The Bucket must be empty! Otherwise it won't work!

.. code-block:: ruby

    AWS::S3::Bucket.delete('my-new-bucket')


.. _S3_Ruby_Forced_Delete_For_Non-Empty_Buckets:

Forced Delete for Non-empty Buckets
-----------------------------------

.. code-block:: ruby

    AWS::S3::Bucket.delete('my-new-bucket', :force => true)


.. _S3_Ruby_Creating_An_Object:

Creating an Object
------------------

This creates a file ``hello.txt`` with the string ``"Hello World!"``

.. code-block:: ruby

    AWS::S3::S3Object.store(
            'hello.txt',
            'Hello World!',
            'my-new-bucket',
            :content_type => 'text/plain'
    )


.. _S3_Ruby_Change_An_Objects_ACL:

Change an Object's ACL
----------------------

This makes the object ``hello.txt`` to be publicly readable, and ``secret_plans.txt``
to be private.

.. code-block:: ruby

    policy = AWS::S3::S3Object.acl('hello.txt', 'my-new-bucket')
    policy.grants = [ AWS::S3::ACL::Grant.grant(:public_read) ]
    AWS::S3::S3Object.acl('hello.txt', 'my-new-bucket', policy)

    policy = AWS::S3::S3Object.acl('secret_plans.txt', 'my-new-bucket')
    policy.grants = []
    AWS::S3::S3Object.acl('secret_plans.txt', 'my-new-bucket', policy)


.. _S3_Ruby_Download_An_Object:

Download an Object (to a file)
------------------------------

This downloads the object ``poetry.pdf`` and saves it in
``/home/larry/documents/``

.. code-block:: ruby

    open('/home/larry/documents/poetry.pdf', 'w') do |file|
            AWS::S3::S3Object.stream('poetry.pdf', 'my-new-bucket') do |chunk|
                    file.write(chunk)
            end
    end


.. _S3_Ruby_Delete_An_Object:

Delete an Object
----------------

This deletes the object ``goodbye.txt``

.. code-block:: ruby

    AWS::S3::S3Object.delete('goodbye.txt', 'my-new-bucket')


.. _S3_Ruby_Generate_Object_Download_URLs:

Generate Object Download URLs (signed and unsigned)
---------------------------------------------------

This generates an unsigned download URL for ``hello.txt``. This works
because we made ``hello.txt`` public by setting the ACL above.
This then generates a signed download URL for ``secret_plans.txt`` that
will work for 1 hour. Signed download URLs will work for the time
period even if the object is private (when the time period is up, the
URL will stop working).

.. code-block:: ruby

    puts AWS::S3::S3Object.url_for(
            'hello.txt',
            'my-new-bucket',
            :authenticated => false
    )

    puts AWS::S3::S3Object.url_for(
            'secret_plans.txt',
            'my-new-bucket',
            :expires_in => 60 * 60
    )

The output of this will look something like::

   http://objects-us-west-1.dream.io/my-bucket-name/hello.txt
   http://objects-us-west-1.dream.io/my-bucket-name/secret_plans.txt?Signature=XXXXXXXXXXXXXXXXXXXXXXXXXXX&Expires=1316027075&AWSAccessKeyId=XXXXXXXXXXXXXXXXXXX

.. _`AWS::S3`: http://amazon.rubyforge.org/
.. _`AWS::S3::Bucket`: http://amazon.rubyforge.org/doc/

.. meta::
    :labels: ruby S3 api
