
# Static Website on S3 with CloudFront and Route53

## Step 1: Create a bucket

#### To create a bucket

1. Sign in to the AWS Management Console and open the Amazon S3 console at

2. Choose Create bucket.

3. Enter the Bucket name (for example, example.com)

4. Choose the Region where you want to create the bucket.

Choose a Region that is geographically close to you to minimize latency and costs, or to address regulatory requirements. The Region that you choose determines your Amazon S3 website endpoint.

5. To accept the default settings and create the bucket, choose Create.

## Step 2: Enable static website hosting

#### To enable static website hosting

1. Sign in to the AWS Management Console and open the Amazon S3 console at

2. In the Buckets list, choose the name of the bucket that you want to enable static website hosting for.

3. Choose Properties.

4. Under Static website hosting, choose Edit.

5. Choose Use this bucket to host a website.

6. Under Static website hosting, choose Enable.

7. In Index document, enter the file name of the index document, typically index.html, but in my case i want to host resume so i have used name as Pankaj_Zire_Resume.pdf

8. Choose Save changes.
* Amazon S3 enables static website hosting for your bucket. At the bottom of the page, under Static website hosting, you see the website endpoint for your bucket.

9. Under Static website hosting, note the Endpoint.

* The Endpoint is the Amazon S3 website endpoint for your bucket. After you finish configuring your bucket as a static website, you can use this endpoint to test your website.

## Step 3: Edit Block Public Access settings

By default, Amazon S3 blocks public access to your account and buckets. If you want to use a bucket to host a static website, you can use these steps to edit your block public access settings.

1. Open the Amazon S3 console

2. Choose the name of the bucket that you have configured as a static website.

3. Choose Permissions.

4. Under Block public access (bucket settings), choose Edit.

5. Clear Block all public access, and choose Save changes.

6. Amazon S3 turns off Block Public Access settings for your bucket. To create a public, static website, you might also have to edit the Block Public Access settings for your account before adding a bucket policy. If account settings for Block Public Access are currently turned on, you see a note under Block public access (bucket settings).

## Step 4: Add a bucket policy that makes your bucket content publicly available

After you edit S3 Block Public Access settings, you can add a bucket policy to grant public read access to your bucket. When you grant public read access, anyone on the internet can access your bucket.

1. Under Buckets, choose the name of your bucket.
 
2.  Choose Permissions.

3. Under Bucket Policy, choose Edit.

4. To grant public read access for your website, copy the following bucket policy, and paste it in the Bucket policy editor.

       {
        "Version": "2012-10-17",
        "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": [
                "s3:GetObject"
            ],
            "Resource": [
                "arn:aws:s3:::Bucket-Name/*"
            ]
        }
       ]
       }


5. Update the Resource to your bucket name.

* In the preceding example bucket policy, Bucket-Name is a placeholder for the bucket name. To use this bucket policy with your own bucket, you must update this name to match your bucket name.

6. Choose Save changes.
* A message appears indicating that the bucket policy has been successfully added.

* If you see an error that says Policy has invalid resource, confirm that the bucket name in the bucket policy matches your bucket name.

* If you get an error message and cannot save the bucket policy, check your account and bucket Block Public Access settings to confirm that you allow public access to the bucket.


## Step 5: Configure an index document (if you want host proper webpage)

When you enable static website hosting for your bucket, you enter the name of the index document (for example, index.html). After you enable static website hosting for the bucket, you upload an HTML file with this index document name to your bucket.

#### To configure the index document

1. Create an index.html file.

* If you don't have an index.html file, you can use the following HTML to create one:

      <html xmlns="http://www.w3.org/1999/xhtml" >
      <head>
      <title>My Website Home Page</title>
      </head>
      <body>
      <h1>Welcome to my website</h1>
      <p>Now hosted on Amazon S3!</p>
      </body>
      </html>

2. Save the index file locally.

* The index document file name must exactly match the index document name that you enter in the Static website hosting dialog box. The index document name is case sensitive. For example, if you enter index.html for the Index document name in the Static website hosting dialog box, your index document file name must also be index.html and not Index.html.

3. In the Buckets list, choose the name of the bucket that you want to use to host a static website.

4. Enable static website hosting for your bucket, and enter the exact name of your index document (for example, index.html).

5. To upload the index document to your bucket, do one of the following:

* Drag and drop the index file into the console bucket listing.

* Choose Upload, and follow the prompts to choose and upload the index file.

## Step 7: Test your website endpoint

After you configure static website hosting for your bucket, you can test your website endpoint.

1. Under Buckets, choose the name of your bucket.

2. Choose Properties.

3. At the bottom of the page, under Static website hosting, choose your Bucket website endpoint,Your index document opens in a separate browser window.

## Speeding up your website with Amazon CloudFront

## Step 1: Create a CloudFront distribution

First, you create a CloudFront distribution. This makes your website available from data centers around the world.

#### To create a distribution with an Amazon S3 origin

1. Open the CloudFront console
2. Choose Create Distribution.
3. On the Create Distribution page, in the Origin Settings section, for Origin Domain Name, enter the Amazon S3 website endpoint for your bucket—for example, example.com.s3-website.us-west-1.amazonaws.com.

CloudFront fills in the Origin ID for you.
4. For Default Cache Behavior Settings, keep the values set to the defaults.

With the default settings for Viewer Protocol Policy, you can use HTTPS for your static website. 
5. For Distribution Settings, do the following:

* Leave Price Class set to Use All Edge Locations (Best Performance).

* Set Alternate Domain Names (CNAMEs) to the root domain and www subdomain. In this tutorial, these are example.com and www.example.com.
* For SSL Certificate, choose Custom SSL Certificate (example.com), and choose the custom certificate that covers the domain and subdomain names.
* In Default Root Object, enter the name of your index document, for example, index.html.

* If the URL used to access the distribution doesn't contain a file name, the CloudFront distribution returns the index document. The Default Root Object should exactly match the name of the index document for your static website. For more information
* Set Logging to On.
* For Bucket for Logs, choose the logging bucket that you created.
* If you want to store the logs that are generated by traffic to the CloudFront distribution in a folder, in Log Prefix, enter the folder name.
* Keep all other settings at their default values.

6. Choose Create Distribution.
7. To see the status of the distribution, find the distribution in the console and check the Status column.

A status of InProgress indicates that the distribution is not yet fully deployed.

After your distribution is deployed, you can reference your content with the new CloudFront domain name.
8. Record the value of Domain Name shown in the CloudFront console, for example, dj4p1rv6mvubz.cloudfront.net.
9. To verify that your CloudFront distribution is working, enter the domain name of the distribution in a web browser.

* If your website is visible, the CloudFront distribution works. If your website has a custom domain registered with Amazon Route 53, you will need the CloudFront domain name to update the record set in the next step.


## Step 2: Update the record sets for your domain and subdomain

Now that you have successfully created a CloudFront distribution, update the alias record in Route 53 to point to the new CloudFront distribution.

####   To update the alias record to point to a CloudFront distribution

1. Open the Route 53 console
2. In the left navigation, choose Hosted zones.
3. On the Hosted Zones page, choose the hosted zone that you created for your subdomain, for example, www.example.com.
4. Under Records, select the A record that you created for your subdomain.
5. Under Record details, choose Edit record.
6. Under Route traffic to, choose Alias to CloudFront distribution. 
7. Under Choose distribution, choose the CloudFront distribution.


8. Choose Save.
9. To redirect the A record for the root domain to the CloudFront distribution, repeat this procedure for the root domain, for example, example.com.
*  The update to the record sets takes effect within 2–48 hours.
10. To see whether the new A records have taken effect, in a web browser, enter your subdomain URL, for example, http://www.example.com.

If the browser no longer redirects you to the root domain (for example, http://example.com), the new A records are in place. When the new A record has taken effect, traffic routed by the new A record to the CloudFront distribution is not redirected to the root domain. Any visitors who reference the site by using http://example.com or http://www.example.com are redirected to the nearest CloudFront edge location, where they benefit from faster download times.

