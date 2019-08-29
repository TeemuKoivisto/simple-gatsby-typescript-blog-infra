# AWS infra for a Gatsby site

The CloudFormation stacks for deploying a Gatsby app with Sceptre on AWS.

# Prerequisites

Requires Python >= 3.5, aws-cli and Sceptre 2.x installed globally: `pip install aws-cli sceptre -U` (or `python3 -m pip install aws-cli sceptre -U` if you have different default Python eg. macOS).

For some reason I couldn't install Sceptre globally with pip so I instead used Brew: `brew install sceptre`.

Also a AWS IAM user with somewhat extensive access privileges. It's easiest to launch the stack locally with admin privileges.

# How to launch

For deploying to your apex domain eg teemukoivisto.xyz, you have in first place buy the domain and have it available in AWS. You should also create a HostedZone for it by hand, since it's more convenient that way. After you have that done, you should also request ACM certificate for your domain.

Then you need to update the following properties in the stack configs:
* `project-code` (you probably want to change it)
* `Domain` set it to apex domain in prod env, subdomain eg dev.teemukoivisto.xyz for your dev env
* `AcmCertArn` TODO what about dev env?
* `HostedZoneId` TODO use created HZ in dev environment

## Manual deployment

Each time you do changes to your edge-lambdas, the flow currently is three-fold:y
1. First you update the lambda, then deploy it.
2. Next you open AWS console and go to the lambda and from `Actions` dropdown you deploy the new version
3. You increment the version number in `cloudfront.yaml` parameters eg to `'2'` and then deploy again
4. Since most of the frontend assets are still in the old CloudFront cache, you have to either invalidate the whole distribution, or do what I think is the easiest way: deploy the Gatsby site again. This way the asset URLs etc change in the `index.html` without you having to mess with the CloudFront.

This flow sucks, I know but since I haven't figured out a better way it is, what it is.

## Launching the stacks

Run simply: `AWS_PROFILE=teemukoivisto.xyz-ci sceptre launch prod` where you should replace the AWS_PROFILE value with an admin AWS user you have configured locally and prod/dev depending on what stack you want to deploy.

You can deploy the stacks one by one using eg `AWS_PROFILE=teemukoivisto.xyz-ci sceptre launch dev/eu-west-1/static-bucket`
