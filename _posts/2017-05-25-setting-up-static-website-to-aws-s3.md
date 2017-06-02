---
title: Setting up static Website on AWS S3
status: false
layout: post
categories: [Aws, s3, cloudfront, route 53 React, React-router]
tags: [Aws, s3, cloudfront, route 53 React, React-router]
published: False
---

Lets go over how to deploy a static website built in React to the cloud using AWS.

### Technologies uses:
1. __AWS Route 53__ - To purchase and configure the domain
2. __AWS S3__ - to host the static website
3. __AWS Cloudfront__ - to cache and serve the static website and also get around the naming limitations of S3
4. __AWS Command Line Tools__ - to do quick deployments to S3 & cache invalidations on Cloudfront
5. __React__, __Redux__, __Webpack__ & __NPM__ - to build the static website

#### Step 1.

We use React to build the static website. Webpack compiles everything and puts it in to the `\dist` folder. In my case this includes html, css, js and font files.
    
![](https://content.screencast.com/users/sandeep45/folders/Jing/media/da39c2c5-a4cd-4d34-9652-3d9d2b092966/00002691.png)

#### Step 2.

Create a bucket on AWS S3. Name it whatever is convenient for you, it doesn't have to match the name of the domain as we are going to be using cloudfront in the middle of the s3 and domain. Enable `Static Website Hosting` on the bucket. This enables auto loading of the index file like `index.html` when the container folder is put in the path.

 ![](https://content.screencast.com/users/sandeep45/folders/Jing/media/6655786e-1622-41fe-b241-7e02e4792b2a/00002693.png)

Copy over the static website i.e. your `\dist` folder to your s3 Bucket. You can do this using the Web GUI. Later we will setup and copy things using command line tools   

![](https://content.screencast.com/users/sandeep45/folders/Jing/media/8e5188c7-c0ab-4dd3-9502-7d39044bf9db/00002694.png)

Now visit your:
- s3 bucket's URL: https://s3.amazonaws.com/quick-calculators/index.html

or

- s3 buckets static website URL: http://quick-calculators.s3-website-us-east-1.amazonaws.com

They will both load your website's `index.html` which you had just copied over to your s3 bucket. Note when using the static website URL, I did not have to specify the `index.html` file in the path.

![](https://content.screencast.com/users/sandeep45/folders/Jing/media/c4837684-51d9-4312-8208-060c4286393c/00002696.png)
 
#### Step 3.

Create a distribution on Cloudfront. Make its `origin` be your s3 static website and specify your domain name with and without `www` as `Altername Domain Names (CNAMEs)`. If you dont have the domain names yet, its best to go to AWS Route 53 and first buy the domain names and then comeback and associate it with your cloudfront distribution. Leave all other settings as-is. By Default it will cache origin files for 24 hours and respond to HTTPS requests with its own SSL cert valid for `*.cloudfront.com`.

![](https://content.screencast.com/users/sandeep45/folders/Jing/media/b4d6f5ca-37d2-49ec-b6c5-792c8901aceb/00002704.png)
 
![](https://content.screencast.com/users/sandeep45/folders/Jing/media/c1eccebf-530f-46af-b356-2bd5c5459334/00002697.png) 

![](https://content.screencast.com/users/sandeep45/folders/Jing/media/586c0bc6-cbaf-46f9-8f8d-e3b0f99b2ae2/00002698.png)

![](https://content.screencast.com/users/sandeep45/folders/Jing/media/2d229d88-0a76-4e71-8ac3-b100ebe14dd6/00002699.png)

Once the cloudfront has finished building the distribution, visit the cloudfront URL and it should load your static website.

![](https://content.screencast.com/users/sandeep45/folders/Jing/media/4d39fb9a-f31b-4602-91bc-e81121418a5f/00002700.png)

#### Step 4.

In Route 53 configure the domain name you want to use for your static website. 

- First, create a `Record Set` for `www` as a `cname` and point it the cloudfront url you had created in Step 3.

![](https://content.screencast.com/users/sandeep45/folders/Jing/media/2e74ca18-d58a-4649-83f8-d68a2dce9b4b/00002701.png)
 
- Second, create another `Record Set` for the naked domain, meaning with nothing infront of it as a `A` record, use the `Alias` option and select the cloudfront URL you had created in Step 3 as value.
 
![](https://content.screencast.com/users/sandeep45/folders/Jing/media/c91cd815-f109-42df-97ff-fb5ddc390d33/00002702.png)

Now visit your domain name with and without www. It will load your static website.
  
![](https://content.screencast.com/users/sandeep45/folders/Jing/media/af5fdcc9-9491-471b-8137-e7305cf7b88e/00002703.png)

#### Step 5.

Install AWS Command Line Tools. On a mac with python present, this is as simple as running `pip install --upgrade --user awscli` on the command line. Detailed Instructions can be found here: http://docs.aws.amazon.com/cli/latest/userguide/installing.html

To deploy your changes to your static website, make your changes, compile them in the to `\dist` folder and then run `aws s3 sync .\dist s3://quick-calculators --profile sandeep45` This will sync contents of the `\dist` folder with the contents of the s3 bucket `quick-calculators`. 

To clear cache of aws cloudfront run `aws cloudfront create-invalidation --distribution-id E2I62KLS7VM16M --paths '/*' --profile sandeep45`. Note to run this command you will need your distribution id. This is useful because by default cloudfront is caching files for 24 hours.

# That's it! Happy AWS static Website Hosting!








 
  