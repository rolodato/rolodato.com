---
layout: post
title: Free static websites with SSL for hackers
---

> Note: [GitLab Pages now fully integrate with Let's Encrypt](https://docs.gitlab.com/ee/user/project/pages/custom_domains_ssl_tls_certification/lets_encrypt_integration.html), making this post obsolete.

This post describes a recipe for setting up a static website for free with these features:

* Custom domains
* SSL with [Let's Encrypt](https://letsencrypt.org/)
* Version control using Git, hosted by [gitlab.com](https://gitlab.com) or your own GitLab instance
* [Jekyll](https://jekyllrb.com/) or other [static site generators supported by GitLab Pages](https://about.gitlab.com/2016/06/17/ssg-overview-gitlab-pages-part-3-examples-ci/)

## Setup

1. [Install Jekyll](https://jekyllrb.com/docs/installation/)
2. Fork [GitLab's template Jekyll repository](https://gitlab.com/pages/jekyll#getting-started), or [create your own locally](https://gitlab.com/pages/jekyll#using-jekyll-locally)
3. [Point DNS records to GitLab](https://docs.gitlab.com/ee/user/project/pages/getting_started_part_three.html#dns-records)

Your site should be accessible through your custom domain using plain HTTP at this point.

## HTTPS setup

To make this part easier, I've written a command-line tool named [gitlab-le](https://github.com/rolodato/gitlab-letsencrypt) which handles obtaining and renewing HTTPS certificates from Let's Encrypt.

Install gitlab-le with `npm install --global gitlab-letsencrypt` (requires [Node.js](https://nodejs.org/en/) 8 or greater).

Test everything works OK with the following command.
See [instructions for getting a GitLab access token](https://docs.gitlab.com/ce/user/profile/personal_access_tokens.html).

{% highlight bash %}
gitlab-le \
  --email YOUR_EMAIL@example.com \
  --domains EXAMPLE.com www.EXAMPLE.com \
  --token YOUR_PERSONAL_GITLAB_ACCESS_TOKEN \
  --repository https://YOUR_REPOSITORY_URL \
  --jekyll \
  --path /
{% endhighlight %}

If everything worked, re-run the same command with an additional `--production` option.

At this point, your website should be accessible via HTTPS. Yay!
However, the certificate we've obtained will only last for 90 days, so we need some way of renewing it.

## Renewing HTTPS certificate

To renew your HTTPS certificate, run gitlab-le again with the same options, including the `--production` option.
Automating this to run every 90 days is left as an exercise for the reader.

> Tip: You can run gitlab-le run without the `--production` option as often as you want without fear of being rate-limited by Let's Encrypt. This allows you to detect and fix issues with your repository before the certificate expiration date.

If you don't want to or can't set this up to run automatically, Let's Encrypt will send notices when your certificate is about to expire to the `--email` used when first obtaining the certificate.

## Forcing HTTPS

Currently it is not possible to automatically redirect all visitors to the HTTPS version of your website.
There is an [open issue on the GitLab CE repository to track this feature](https://gitlab.com/gitlab-org/gitlab-ce/issues/28857).

As a temporary workaround, you can redirect using JavaScript on your website:

{% highlight js %}
if (location.protocol !== "https:") location.protocol = "https:";
{% endhighlight %}

## Alternatives

[GitHub Pages](https://pages.github.com/) offer these same features but SSL on custom domains is provided with [CloudFlare's Universal SSL](https://blog.cloudflare.com/secure-and-fast-github-pages-with-cloudflare/), which is not true end-to-end encryption.

[Netlify](https://www.netlify.com/)'s free plan offers these same features with a friendly UI, but it is a closed-source service.
