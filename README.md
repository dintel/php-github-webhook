PHP GitHub webhook package
==========================

## Overview ##

This package contains classes that allow keeping up to date git repository by
performing `git pull` on every `git push` to your GitHub repository. To achieve
this you have to add simple script to your git repository and configure webhook
on your GitHub repository.

## Webhook script ##

For a webhook script to work you have to do 3 things:
1. Add this package to your composer dependencies
2. Add a simple PHP script that will actually handle calls to webhook
3. Configure your GitHub repository webhook to be called every time commits are
   pushed to GitHub

### Composer ###

To add dependency, simply edit your `composer.json` and add to your `require`
block following dependency - `"dintel/php-github-webhook": "0.1.*"`. This is the
simplest way to bring php-github-webhook libraries into your project.

### Webhook script ###

To actually handle webhook calls, you have to add PHP script that will be
accessible publicly and it will run `git pull` every time GitHub calls it.
A simplest example of such webhook script:
```php
<?php
require(__DIR__ . "/vendor/autoload.php");

use GitHubWebhook\Handler;

$handler = new Handler("<your secret>", __DIR__);
if($handler->handle()) {
    echo "OK";
} else {
    echo "Wrong secret";
}
```

In the script above `<your secret>` should be some random string you choose and
it should be later supplied to GitHub when defining webhook. For more
information about secrets and how they are used in GitHub webhook read
[Webhooks | GitHub API](https://developer.github.com/webhooks/).

NOTE: Since this script has sensitive data in it (`secret` that is used to
validate requests), it is advised to not put that script under git control by
excluding it in `.gitignore` file. Another option is to take `secret` from some
environment variable that would be defined by other means (like `SetEnv` in
apache configuration).

### GitHub repository configuration ###

To set up a repository webhook on GitHub, head over to the **Settings** page of your
repository, and click on **Webhooks & services**. After that, click on **Add webhook**.

Fill in following values in form:
* **Payload URL** - Enter full URL to your webhook script
* **Content type** - should be "application/json"
* **Secret** - same secret you pass to constructor of `Handler` object
* Webhook should receive only push events and of course be active

Click **Add webhook** button and that's it.

## Classes ##

### Handler ###

Handler class actually handles webhook calls. It first checks GitHub signature
and then executes `git pull` if signature and secret match.

Here is a complete list of methods:
* **\_\_construct($secret, $gitDir, $remote = null)** - Constructor. Constructs new
  webhook handler that will verify that requests coming to it are signed with
  `$secret`. `$gitDir` must be set to path to git repo that must be updated.
  Optional `$remote` specifies which remote should be pulled.
* **getData()** - Getter. After successful validation returns parsed array of data
  in payload. Otherwise returns `null`.
* **getDelivery()** - Getter. After successful validation returns unique delivery
  number coming from GitHub. Otherwise returns `null`.
* **getEvent** - Getter. After successful validation returns name of event that
  triggered this webhook. Otherwise returns `null`.
* **getGitDir()** - Getter. Returns `$gitDir` that was passed to constructor.
* **getGitOutput** - Getter. After successful validation returns output of git
  as array of lines. Otherwise returns `null`.
* **getRemote()** - Getter. Returns `$remote` that was passed to constructor.
* **getSecret()** - Getter. Returns `$secret` that was passed to constructor.
* **handle()** - Handle the request. Validates that incoming request is signed
  correctly with `$secret` and executes `git pull` upon successful validation.
  Returns `true` on succes or `false` if validation failed.
* **validate()** - Validate request only. Returns boolean that indicates whether
  the request is correctly signed by `$secret`.
