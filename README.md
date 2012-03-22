# Adserver JSON Feed

This projects aims to describe the JSON feed one can consume from the
[adcloud](http://adcloud.com/) adsever. The following topics will be
covered:

 1. What does the JSON feed look like? This will be described in JSON
    Schema.
 2. How can you interact with the API?

The compiled guides are deployed as GitHub Pages and can be found under
the following URL: <http://adcloud.github.com/adserver-json-feed>.

## Usage

In order to add or edit content for this guide you need Ruby and Bundler
installed. With bundler installed just run the following command in the
project root:

    $ bundle install

This will install the [guides](https://github.com/wycats/guides) gem
with all its dependencies. After you've done that just run this command:

    $ bundle exec guides preview

You will have a preview the rendered guides available at
<http://localhost:9292>.

When you're ready to deploy the latest changes as GitHub Pages you have
to perform the following commands:

    $ bundle exec guides build
    $ bundle exec rake push

**Reminder:** If this is your first deployment of the guides you have to
setup the deployment first:

    $ bundle exec rake setup
