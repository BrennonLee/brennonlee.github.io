---
layout: post
title: Enable CORS For S3 Static Sites
---

This post is to help configure CORS (Cross Origin Resource Sharing) on AWS S3 for Static Sites.

#### Prerequisites
- Have an AWS account (we will only be using services under the free tier so don't worry about this costing you any 💰).
- You are logged into the AWS Console.

## Why would I do this?
 Configuring Cross Origin Resource Sharing is a great way to organize your website. This allows you to store all your JS files in one bucket, images in another, and then load the content that's needed into your html bucket.

## Let's Get Started
First, we need to create a bucket that will host our static site. Head over to the Amazon S3 and create a bucket like so:

![_config.yml]({{ site.baseurl }}/images/2020-07-10/create-bucket.png)

For simplicity of this tutorial, go ahead and uncheck "Block all public access". You should see a banner popup that makes you confirm this action. This is going to make this bucket publicly accessible (this is needed so we can make the files we'll upload later on available).

![_config.yml]({{ site.baseurl }}/images/2020-07-10/ack-public-access.png)

Once your bucket is created, we want our bucket to host our static website.
To set that up, navigate over to "Properties" -> "Static website hosting" -> and check "Use this bucket to host a website". A few fields will popup asking which HTML files to use and we'll just stick with the defaults.

![_config.yml]({{ site.baseurl }}/images/2020-07-10/enable-static-hosting.png)

Now we need to upload the actual `index.html` and `error.html` files. You can copy/paste the contents down below and upload them:
```
<!-- index.html -->
<script src="https://code.jquery.com/jquery-1.11.0.min.js"></script>
<script src="https://code.jquery.com/jquery-migrate-1.2.1.min.js"></script>

<html>
  <body>
    <h1>This is index.html</h1>
    <br>
    <h3>This is the rest of the page</h3>
  </body>
</html>

```

```
<!-- error.html -->
<html>
  <body>
    <h1>
      Uh oh, looks like something went wrong :/
    </h1>
  </body>
</html>

```

Go ahead and upload these files to the bucket we just created. Make sure when you do, to select "Grant public read access to this object(s)".

To test our static site is serving the correct files, you can find the endpoint under your bucket's "Properties" settings.
Find the URL located at "Properties" -> "Static website hosting".
Test that out in the browser and you should see the following content:

![_config.yml]({{ site.baseurl }}/images/2020-07-10/part-1-content.png)

Now that we got our static site up and running, let's create another bucket where we will upload different files but load them in this site.


Go ahead and create your second bucket (naming it whatever you want, but it must be globally unique). I named mine "bucket-for-brennons-other-files" and be sure to make this is publicly accessible as well. Also mark this site as "Static website hosting" just like we did for our first bucket.

For this new bucket, upload the following html file (make sure it's public!):
```
<!-- restOfContent.html -->
  <br>
  <h2>This content was loaded from another S3 bucket!</h2>
  </body>
</html>

```

Next, we need to modify our original `index.html` file to load the content from our new bucket.
To do so, we are going to take note of the endpoint of our new bucket. Do this just
like you did before and copy that endpoint to your clipboard.
Open and modify your original `index.html` file with the following content:

 ```
 <!-- index.html -->
 <script src="https://code.jquery.com/jquery-1.11.0.min.js"></script>
 <script src="https://code.jquery.com/jquery-migrate-1.2.1.min.js"></script>

 <html>
   <body>
     <h1>This is index.html</h1>
     <br>
     <h3>This is the rest of the page</h3>
     <div id="file-from-other-s3-bucket"></div>

     <script>
      // Put your new bucket's endpoint here
      $("#file-from-other-s3-bucket").load("<YOUR_NEW_BUCKETS_ENDPOINT>/restOfContent.html")
     </script>

 ```

Replace the `index.html` file in your original bucket with this modified version above.

As of right now, we will get a CORS error by visiting our original site. Before this will work, we need to enable the CORS rules for our new bucket.

Navigate to your new bucket and go to "Permissions" -> "CORS configuration" and paste the content below (with your modifications):

```
<?xml version="1.0" encoding="UTF-8"?>
<CORSConfiguration xmlns="http://s3.amazonaws.com/doc/2006-03-01/">
<CORSRule>
    <AllowedOrigin>YOUR_ORIGINAL_BUCKETS_ENDPOINT</AllowedOrigin>
    <AllowedMethod>GET</AllowedMethod>
    <MaxAgeSeconds>3000</MaxAgeSeconds>
    <AllowedHeader>Authorization</AllowedHeader>
</CORSRule>
</CORSConfiguration>

```

Hit Save and then reloading your site should successfully load the other file!

![_config.yml]({{ site.baseurl }}/images/2020-07-10/finished-site.png)

## Congratulations! 🎉
You just setup a static site that uses CORS to load files from separate S3 buckets! This tutorial was a simple example but at large, this is an effective way to keep your static sites organized and easy to navigate.
