+++
title = "Setting Up Continuous Deployment for this Blog"
date = "2025-01-16"
+++

When I first start building infrastructure on AWS, I prefer to manually create everything first. It helps me to sort out the pieces in my brain and also reduces errors once I move to CloudFormation templates.

Today, I'm creating the initial infrastructure setup for this blog. Eventually, I want this to be much more automated. My vision is to have a CloudFormation template that creates the CodePipeline and any necessary infrastructure. The CodePipeline will trigger on a push to `main` in my Github repo. Then it will build the Hugo project and spit out the `public` folder into an s3 bucket.

For now, I'm going to manually create the CodePipeline in the AWS UI without any build step. My process is going to look like this: write a new post, build locally, and
push to `main`. The CodePipeline will then run automatically and output the content to s3.

I'll need to create a few things in AWS to get this setup working:

- An s3 bucket to host my static website
- A Github connection for the CodePipeline
- The CodePipeline itself

## S3 Bucket

First, I head to s3 and create a new bucket. I leave mostly everything default, but I do need to uncheck "Block All Public Access" to allow people to access it. I go into the newly created bucket and head over to properties and turn on "Static website hosting" with the index document as `index.html`. This will be the entrypoint for my blog, so I need to make sure there's an `index.html` in the root of my Github repo.

```
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
                "arn:aws:s3:::${Bucket-Name}/*"
        	]
    	}
    ]
}
```

The last piece is setting up the bucket policy under the "Permissions" section of the bucket. I allow `s3:GetObject` access for all (`*`) on my specific bucket (`${Bucket-Name}`). This will ensure the files that make up my blog are accesible to anyone viewing my site.

## Github Connection

The next step is to create the Github connection for the CodePipeline to use. I head over to CodePipeline, then settings, connections, and select "Create Connection". I select "Github" as the provider and name it to match my Github account. Then I hit "Connect to Github" and "Install a new app". From there, I select my Github account and repos to allow access to. Once I'm done, I select "Install & Authorize". After I return to AWS, I select "Connect". Now that my Github connection is ready to use, I can create the CodePipeline.

## CodePipeline

I go to CodePipeline and select "Create pipeline" and then "Build custom pipeline". I name my pipeline, leaving all other settings default and then select "Next". From here, the first step is to setup the Source to be my blog's Github repo. I select Github (via Github App) as my source provider and then select the connection I just created, my website repo, and the default branch of `main`. I leave everything else as default and then select "Next". I hit "Skip build stage" and then "Skip test stage". From there I select "S3" as my deploy provider, select my bucket I created, and then check the "Extract file before deploy" setting. Then I hit "Next" and "Create Pipeline".

## Testing

From here the CodePipeline automatically runs and uploads my website to the bucket. I go to my S3 bucket and find "Static website hosting" under "Properties" and get my "Bucket website endpoint". This is where my website is being hosted. After everything looks good at that URL, I test a push to `main` in my Github repo and I can see the pipeline running and updating. Success!

## Almost There

Now that everything's working, I want my blog to use my custom domain and I would like it to use https. The one downside about the static hosting feature on s3 is that it doesn't support https. In order to do this, I need to create a certificate in Certificate Manager and then create a Cloudfront distribution with my s3 bucket as the origin.

I request a certificate from Certificate Manager for my domain using DNS validation. Then I add the CNAME record into my DNS to validate that I own the domain. Once that all clears, I'm ready to create my Cloudfront distribution. I set the "Origin Domain" to my s3 bucket, making sure to use the website endpoint I tested with earlier. I update the "Alternate domain name (CNAME)" to my domain and then select the ACM certificate that I just created from the list under "Custom SSL certificate". I set the caching policy to "CachingDisabled" and select "Redirect HTTP to HTTPS" for protocol policy. This ensures that any changes to the repo are immediately displayed on my site. I leave everything else as default and select "Create distribution".

After about 10-20 minutes, my distribution is deployed and I can now access my blog using my custom domain. All done for today!
