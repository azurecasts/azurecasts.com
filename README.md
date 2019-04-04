# AzureCasts
Screencasts, but for Azure

## The Public Site

This is the public site for azurecasts.com. 

## Install and Running

You can run the site by cloning this repo and then running Jekyll locally. This is easier said than done, but what works for me is:

 - Install rbenv using `brew install rbenv` (if you're on a Mac - don't know if it runs on Windows)
 - Install Ruby 2.6.1 using `rbenv install 2.6.1` and install Bundler using `gem install bundler`.
 - Set the local Ruby in the cloned directory using `rbenv local 2.6.1`.

Once that's done you can install the required gems using Make: `make gems`. This will drop the gems into a local directory so everything is nice and tidy.

If this fails or you're on Windows, maybe just use Docker.