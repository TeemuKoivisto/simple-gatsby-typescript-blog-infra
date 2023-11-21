# Simple Gatsby Site Infrastructure

AWS infrastructure for my [Gatsby site](https://github.com/TeemuKoivisto/simple-gatsby-typescript-blog).

Created using these tutorials, code:
* https://medium.com/@thetrevorharmon/how-to-make-a-super-fast-static-site-with-gatsby-typescript-and-sass-3742c00d4524
* https://www.gatsbyjs.org/tutorial/

## Requirements

Requires Python >= 3.5, aws-cli and sceptre installed globally: `pip install aws-cli sceptre -U` (or `python3 -m pip install aws-cli sceptre -U` if you have different default Python eg. macOS).

For some reason I couldn't install Sceptre globally with pip so I instead used Brew: `brew install sceptre`.

## How to run the stacks

You should have an AWS account with *admin* permissions. Yes, scary I know but eh...

Using sceptre however is super simple, for creating the stack you'll need only to launch it with:
1. `sceptre launch-env <env>` where env could be `dev`.
2. Wait until the stack creation finishes.

## gatsby-blog-us-east

Since you can only create Edge Lambdas in us-east-1 region I created a separate sceptre stack for it.

References:
* https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/lambda-edge-how-it-works-tutorial.html
* https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/lambda-event-structure.html

```
 headers['strict-transport-security'] = [{key: 'Strict-Transport-Security', value: 'max-age= 63072000 
 ; includeSubdomains; preload'}]; 
 headers['content-security-policy'] = [{key: 'Content-Security-Policy', value: "default-src 'none'; img-src 'self'; script-src 'self'; style-src 'self'; object-src 'none'"}]; 
 headers['x-content-type-options'] = [{key: 'X-Content-Type-Options', value: 'nosniff'}]; 
 headers['x-frame-options'] = [{key: 'X-Frame-Options', value: 'DENY'}]; 
 headers['x-xss-protection'] = [{key: 'X-XSS-Protection', value: '1; mode=block'}]; 
 headers['referrer-policy'] = [{key: 'Referrer-Policy', value: 'same-origin'}]; 
```

## gatsby-blog

The default resources that are created to the eu-west-1 region.

### Order of creating the stacks

0. Buy your domain. Add either your domain or a new subdomain name to `gatsby-blog/config/staging/dns.yaml`.
0.1 Create your goddamn buckit with the Gatsby assets???
1. Launch dns: `cd gatsby-blog && sceptre launch-stack staging dns`.
2. Go to AWS console, add your subdomain and SSL certificate manually [instructions](https://github.com/TeemuKoivisto/-simple-gatsby-typescript-blog-infra/tree/master/gatsby-blog/manual-stacks).
3. Launch edge-lambda stack in us-east-1: `cd gatsby-blog-us-east && sceptre launch-stack staging edge-lambda`.
4. Copy and paste your S3 static website URL, ACM certificate ARN and Edge Lambda ARN to `gatsby-blog/config/staging/cloudfront.yaml`.
5. Launch cloudfront: `cd gatsby-blog && sceptre launch-stack staging cloudfront`. This will take a while (like ~30 min).
6. HMM... ready? Visit your website! 

NOTE: `sceptre delete-env` (or `delete-stack`) won't work for some of the resources...
At least:
* HostedZones (which actually is a really good thing)

## Sceptre gotchas

Don't use `.yml`-files, always `.yaml`. Sceptre won't read those by default (unless specified explicitly).

## CloudFormation gotchas

* `Ref` is for variable only: `Name: !Ref Domain`
* `Sub` is for variable in a string: `Name: !Sub ${Domain}-lambda`
