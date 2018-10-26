# Manual stacks for the Gatsby Blog

The manually created stacks that weren't automated due to them being one-time jobs / easier to do in console.

# Create subdomain for your main site

If you're going to use `staging.<yoursite.com>` as your staging site you'll need to configure some name server settings by hand.

You'll need to have created the `dns.yaml` stack first: `sceptre launch-stack staging dns`. Then:

1. Go to your Route 53 page, open your staging-site's configuration
2. Copy the name servers of your site, there should be 4 of them named eg. `ns-1228.awsdns-25.org.`
3. Return and go to your main site's configuration, click `Create Record Set`
4. Select `NS` record and paste the addresses of your staging server's name servers. Set `staging` as its name.
5. Save the record, now you'll need to wait for a while. Look up your site's status with `dig subdomain.example.com`

Reference:
* https://stackoverflow.com/questions/35778315/can-i-have-a-route53-subdomain-in-a-different-hosted-zone

# SSL Certificate with ACM

So as you might know you need a SSL certificate for serving your site through HTTPS. This is mandatory.

You might have used Let's Encrypt or some other SSL provider but I'm afraid I can't help with those certificates.

With AWS you can use AWS Certificate Manager to issue certificates with an automatic 30 day renewal. They are free at least in this use case so try them out!

1. Go to your ACM page: https://console.aws.amazon.com/acm/
2. Click `Request Certificate`, choose public, enter your domain name, choose defaults for the next two pages
3. Once you see `Request in progress` click your domain, click `Create record in Route 53`
4. Now you're going to have to wait for a while (could be as long as 30 minutes). Once you're done you can add the **arn** of the certificate to your CloudFront-stack

