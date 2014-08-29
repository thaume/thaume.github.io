---
layout: post
title: NodeJS streaming files to Amazon S3
date: 2014-02-12 16:16:51.000000000 +01:00
type: post
published: true
meta:
  seo_title: NodeJS streaming files to Amazon S3
  seo_description: This blog post shows a technique to upload files directly to amazon S3 without having to create temporary files on the system.
  seo_keywords: nodejs, aws, knox, streams, busboy
  seo_robots_index: '0'
  seo_robots_follow: '0'
author:
  login: Tom
  email: tom.coquereau@gmail.com
  display_name: Tom Coquereau
---

In this blog post I want to take a look at a way to upload files directly to amazon S3 without having to create temporary files on the system. I will use [knox](https://github.com/LearnBoost/knox) (a nodejs client to upload objects to amazon S3) and [busboy](https://github.com/mscdex/busboy) (a fast streaming parser for HTML form data). You will find the final code of this post in [this github repo](https://github.com/thaume/s3-streaming).

## Setting up the ExpressJS app

### Installing dependencies

After creating your Express project you'll need to run these commands in your terminal to get all the modules and set up our file where we'll handle file upload (remember to run these commands from the root directory fo your newly created express app).

{% highlight bash %}
$ npm install

$ npm install --save busboy // data form parser

$ npm install --save connect-busboy // busboy middleware

$ npm install --save knox // S3 client

$ mkdir /services/

$ touch /services/aws-streaming.js
{% endhighlight %}

### Get our app to communicate with S3

First you'll need to set your S3 credentials in order for our app to be able to communicate with your S3 bucket. They belong in the config file, which is in `app/config/development.json` and looks like this:

{% highlight javascript %}
{
  "aws": {
    "AWS_ACCESS_KEY_ID": "YOUR_AWS_ACCESS_KEY_ID",
    "AWS_SECRET_ACCESS_KEY": "YOUR_AWS_SECRET_ACCESS_KEY",
    "S3_BUCKET_NAME": "YOUR_S3_BUCKET_NAME"
  }
}
{% endhighlight %}

We'll use this config file later in our file handler function.

## Setting up our route in Express

We will only need one route that will both handle the GET and POST request from our webview. This is just for the sake of making something simple, you shouldn't (in my opinion) use such a route for a production app. In `routes/index.js` we have:

{% highlight javascript %}
var awsUpload = require('../services/aws-streaming');

// Index route
// ===========
exports.index = function(req, res){
  if (req.method === 'POST') {
    return awsUpload(req, function(err, url) {
      res.end('<html><head></head><body>\
                 <h1>All good !</h1>\
              </body></html>');
    });
  }

  res.writeHead(200, { Connection: 'close' });
  res.end('<html><head></head><body>\
             <form method="POST" enctype="multipart/form-data">\
              <input type="file" name="filefield"><br />\
              <input type="submit">\
            </form>\
          </body></html>');
};
{% endhighlight %}

Our index route responds with an html form if the request is not a POST. If the request is a POST, it will first upload the file sent from the browser to S3 and then render a success message to the webview.

## Parsing a form-data with Busboy

The app is now ready to send our file to S3. If you try to send a file right now, you will get nothing, even if you try to inspect the 'req.body' property (supposed to contain what was sent from the browser). That is because you need to parse the content sent from the browser, to do that we'll use Busboy which handles files upload.

### Setting the Busboy middleware

In our `app.js` file, we set a couple of things: routes, middlewares, app variables and we also start the server from here. What's interesting to us right now is the middleware stack:

{% highlight javascript %}
var busboy = require('connect-busboy');

...

app.use(busboy());
app.use(express.methodOverride());
app.use(app.router);
{% endhighlight %}

In case you followed the '[Do not use the bodyParse in express](http://andrewkelley.me/post/do-not-use-bodyparser-with-express-js.html)' series of posts, the main difference with the bodyParse middleware is that Busboy will never write temporary files to the filesystem. The problem with writing files to the filesystem on every file upload is that you can overwhelm your server with useless temporary files. Busboy uses streams2 all the way and doesn't block your filesystem. The fact that it doesn't do I/O means better performances too.

Here we set busboy as a middleware ('busboy-connect') so that it handles every file upload that comes toward the server. It means that it will create streams for each file, you will be able to listen to this streams as you'll find later in this blog post.

### Handling files

Now we need to get the file and send it to S3. Sounds easy since Busboy does handle the whole thing right?

Well, there's still a little work to be done: S3 needs a 'content-length' header to be set otherwise the upload will not complete. Streams by definition do not have 'content-length' or length. Which means we need to find the streams length before sending it over to S3. The strategy I use here is simple: buffer the stream to get its length.

{% highlight javascript %}
req.busboy.on('file', function(fieldname, file, filename, encoding, mimetype) {
    if (!filename) {
      // If filename is not truthy it means there's no file
      return;
    }
    // Create the initial array containing the stream's chunks
    file.fileRead = [];

    file.on('data', function(chunk) {
      // Push chunks into the fileRead array
      this.fileRead.push(chunk);
    });

    file.on('error', function(err) {
      console.log('Error while buffering the stream: ', err);
    });

    file.on('end', function() {
      // Concat the chunks into a Buffer
      var finalBuffer = Buffer.concat(this.fileRead);

      req.files[fieldname] = {
        buffer: finalBuffer,
        size: finalBuffer.length,
        filename: filename,
        mimetype: mimetype
      };

    });
  });
{% endhighlight %}

The resulting object (req.files[fieldname]) now features all the requirements of S3.

### Unique filename

Before sending the file, let's imagine that we want to have a strategy to always have different file names (if you send two files with the same file name, S3 will erase the older one). With the following technique, you will always have different file names:

{% highlight javascript %}
// Generate date based folder prefix
var datePrefix = moment().format('YYYY[/]MM');
var key = crypto.randomBytes(10).toString('hex');
var hashFilename = key + '-' + filename;

var pathToArtwork = '/artworks/' + datePrefix + '/' + hashFilename;
{% endhighlight %}

We'll now save files in a month and a day folder, following this pattern: `/artwork/[MONTH]/[DAY]/[random-number]filename`. This might not sound so important, but you need to know that S3 is much more efficient when you create a hierarchy of files rather than just putting all files at the same level and if you want to scale a bit, you'll need random naming to avoid files erasing others.

### Setting headers

We then need to fix the headers for the request to be valid for S3 ('x-amz-acl' makes the image readable by anyone with its URL, that's what you want to use for most images):

{% highlight javascript %}
var headers = {
    'Content-Length': req.files[fieldname].size,
    'Content-Type': req.files[fieldname].mimetype,
    'x-amz-acl': 'public-read'
};
{% endhighlight %}

Once the headers are set, we have all the data and information needed to push our file to our S3 bucket.

### Sending files to S3

To put our newly created Buffer to S3, we'll need to use Knox at the end of our `req.busboy.on('file')` callback (after the headers setup):

{% highlight javascript %}
Knox.aws.putBuffer( req.files[fieldname].buffer, pathToArtwork, headers, function(err, response){
  if (err) {
    console.error('error streaming image: ', new Date(), err);
    return next(err);
  }
  if (response.statusCode !== 200) {
    console.error('error streaming image: ', new Date(), err);
    return next(err);
  }
  console.log('Amazon response statusCode: ', response.statusCode);
  console.log('Your file was uploaded');
  next();
});
{% endhighlight %}

Here we use the putBuffer method of Knox [https://github.com/LearnBoost/knox#put](https://github.com/LearnBoost/knox#put)) and we then call `next()` once S3 responds with a 200 status code and... voilà! Check your S3 bucket, your file should be there ready to be used!

You will find the final code of this post in [this github repo](https://github.com/thaume/s3-streaming).
