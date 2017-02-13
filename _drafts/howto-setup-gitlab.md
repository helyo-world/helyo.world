---
title: How to set up your own GitLab server
layout: post
author: Corentin
cover: /images/tools-rack.jpg
tags:
- tools
---

One thing developers are pretty much obliged to use nowadays is Git. So why not use your own server?

**Note**: This is the first post of this serie, aimed at helping you set up a full dev environment with GitLab, Docker and some other tools.

---

* TOC
{:toc}

---

# But wait… Why should I use my own Git server?

This may have been the first question that came to your mind when you saw this title. And you're completely right to ask yourself that.

There are many reasons why having your own Git server is useful, but keep in mind that it all depends on you: if you don't feel like you need one after reading a few lines of this post, then you might not need it. And that's okay!

Anyway, here are some of the reasons why I installed my very own private Git server, that I now share with Éric:

* It's (almost) **free**. You just need to pay for the server itself
* It's **private**. Nobody else can access your repos and data
* It's **self-managed**. You don't depend on an external provider (except, maybe, your hosting company)
* It's **easy**. There is almost nothing to manage on a regular basis, and the setup process is fairly accessible
* It's **complete**. There are already a bunch of options on GitLab that you can find on other providers like GitHub or Bitbucket, like:
	* Issue tracking
	* Webhooks
	* Teams & User access
	* Continuous development

# OK, I'm sold. What do I need?

Well, first you need to have your own server to host GitLab.

You should also make sure that it is powerful enough to make [GitLab CE](https://about.gitlab.com/downloads/) work. Check out [this page](https://docs.gitlab.com/ce/install/requirements.html) to know all about the requirements.

In this case, I have been using a Ubuntu 16.04 system, with 4 cores and 4 GB of RAM and a bunch of storage. The CPU capacity is enough, though the RAM is a little short if you want to use the same server for other things (CI, Docker deployment, regular hosting, etc).

The same server is already used for other projects, served by nginx, so I will reuse it.

# All checks are green. How do I install that?

Well, no better way to do it than by following [the docs](https://about.gitlab.com/downloads/#ubuntu1604):

```bash
sudo apt install curl openssh-server ca-certificates postfix
curl -sS https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh | sudo bash
sudo apt install gitlab-ce
```

You now have a shiny GitLab server, ready to be started and used. But if you already have a web server on the same machine, you might need to configure it to redirect to the GitLab Workhorse.

You can stop there if you only have GitLab of your machine, at il will listen on port 80 by default. Just execute the `sudo gitlab-ctl reconfigure` command and you'll be good to go.

# Great. So how do I use nginx as a proxy?

First, check that you have Passenger installed. If not, you should install it alongside nginx, it's really helpful.

Then, you may create a file for your site in the nginx directory. Let's call it `gitlab.helyo.world`.

```bash
sudo nano /etc/nginx/sites-availables/gitlab.helyo.world
```

*Yes, I use nano. Not vim. Not emacs. Good ol' nano. Deal with it.*

## Configure the socket to use the GitLab Workhorse

```nginx
upstream gitlab-workhorse {
	server unix://var/opt/gitlab/gitlab-workhorse/socket fail_timeout=0;
}
```

## Configure the site in nginx

```nginx
server {
	listen 80;
	# listen [::]:80; # Uncomment if you use IPv6

	server_name gitlab.helyo.world;

	root /opt/gitlab/embedded/service/gitlab-rails/public;

	access_log /var/log/nginx/gitlab.helyo.world.access.log;
	error_log /var/log/nginx/gitlab.helyo.world.error.log;

	client_max_body_size 500m;

	passenger_ruby /opt/gitlab/embedded/bin/ruby;
	passenger_env_var PATH "/opt/gitlab/bin:/opt/gitlab/embedded/bin:/usr/local/bin:/usr/bin:/bin";

	passenger_user git;
	passenger_group git;

	passenger_enabled on;
	passenger_min_instances 1;

	location ~ ^/[\w\.-]+/[\w\.-]+/(info/refs|git-upload-pack|git-receive-pack)$ {
		# 'Error' 418 is a hack to re-use the @gitlab-workhorse block
		error_page 418 = @gitlab-workhorse;
		return 418;
	}

	location ~ ^/[\w\.-]+/[\w\.-]+/repository/archive {
		# 'Error' 418 is a hack to re-use the @gitlab-workhorse block
		error_page 418 = @gitlab-workhorse;
		return 418;
	}

	location ~ ^/api/v3/projects/.*/repository/archive {
		# 'Error' 418 is a hack to re-use the @gitlab-workhorse block
		error_page 418 = @gitlab-workhorse;
		return 418;
	}

	# Build artifacts should be submitted to this location
	location ~ ^/[\w\.-]+/[\w\.-]+/builds/download {
		client_max_body_size 0;
		# 'Error' 418 is a hack to re-use the @gitlab-workhorse block
		error_page 418 = @gitlab-workhorse;
		return 418;
	}

	location @gitlab-workhorse {
		## https://github.com/gitlabhq/gitlabhq/issues/694
		## Some requests take more than 30 seconds.
		proxy_read_timeout		3600;
		proxy_connect_timeout	300;
		proxy_redirect			off;

		# Do not buffer Git HTTP responses
		proxy_buffering off;

		proxy_set_header	Host				$http_host;
		proxy_set_header	X-Real-IP			$remote_addr;
		proxy_set_header	X-Forwarded-For	$proxy_add_x_forwarded_for;
		proxy_set_header	X-Forwarded-Proto   $scheme;

		proxy_pass http://gitlab-workhorse;

		## The following settings only work with NGINX 1.7.11 or newer
		#
		## Pass chunked request bodies to gitlab-workhorse as-is
		# proxy_request_buffering off;
		# proxy_http_version 1.1;
	}


	## Enable gzip compression as per rails guide:
	## http://guides.rubyonrails.org/asset_pipeline.html#gzip-compression
	## WARNING: If you are using relative urls remove the block below
	## See config/application.rb under "Relative url support" for the list of
	## other files that need to be changed for relative url support
	location ~ ^/(assets)/ {
		root /opt/gitlab/embedded/service/gitlab-rails/public;
		gzip_static on; # to serve pre-gzipped version
		expires max;
		add_header Cache-Control public;
	}

	error_page 404 /404.html;
	error_page 422 /422.html;
	error_page 500 /500.html;
	error_page 502 /502.html;
	error_page 503 /503.html;

	location ~ ^/(404|422|500|502|503)\.html$ {
		root /opt/gitlab/embedded/service/gitlab-rails/public;
		internal;
	}
}
```

## Configure GitLab to work behind nginx

You will need to open the `/etc/gitlab/gitlab.rb` file to change a few lines:

```ruby
external_url 'http://gitlab.helyo.world'
gitlab_rails['internal_api_url'] = 'http://gitlab.helyo.world/'
```

```ruby
unicorn['enable'] = true;
```

```ruby
web_server['external_users'] = ['www-data']
```

```ruby
nginx['enable'] = false
```

Then tell GitLab to apply these changes:

```bash
sudo gitlab-ctl reconfigure
```

And you should be good to go! You can now visit your domain and get started. You will be asked to configure the *root* account's password befor you can log in.

# What's next?

Well, if you want to have a secure environment, you should consider switching to HTTPS. So we'll see how to do that very soon!

[CI]: Continuous Integration
