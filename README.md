Lobot: Your Chief Administrative Aide on Cloud City
============================

![Lobot](http://i.imgur.com/QAkd7.jpg)
###A "one click" solution for deploying CI to EC2

Lando Calrissian relies on Lobot to keep Cloud City afloat, and now you can rely on Lobot to keep your continuous integration server running in the cloud. Lobot is a gem that will help you spin-up, bootstrap, and install Jenkins for CI for your Rails app on Amazon EC2.

# What do I get?

* rake tasks for starting a CI instance
* capistrano tasks for bootstrapping and deploying to an EC2 instance
* chef recipes for configuring a Centos server to run Jenkins and build Rails projects.

all you'll need to do is run the following commands:

    rails g lobot:install
    rails g lobot:config
    rake ci:server_start
    cap ci bootstrap
    cap ci chef

Read on for an explanation of what each one of these steps does.

## Install

Add lobot to your Gemfile, in the development group:

    gem "lobot", :group => :development

## Generate
Lobot is a Rails 3 generator.  Rails 2 can be made to work, but you will need to copy the template files into your project.

    rails g lobot:install

## Setup
You can use a generator to interactively generate the config/ci.yml that includes some reasonable defaults and explanations

    rails g lobot:config

Alternatively, manually edit config/ci.yml

    ---
    app_name: # a short name for your application
    app_user: # the user created to run your CI process
    git_location: # The location of your remote git repository which Jenkins will poll and pull from on changes.
    basic_auth:
    - username: # The username you will use to access the Jenkins web interface
      password: # The password you will use to access the Jenkins web interface
    credentials:
      aws_access_key_id: # The Access Key for your Amazon AWS account
      aws_secret_access_key: The Secret Access Key for your Amazon AWS account
      provider: AWS # leave this one alone
    server:
      name: run 'rake ci:server_start to populate'
      instance_id: run 'rake ci:server_start to populate'
    build_command: ./script/ci_build.sh
    ec2_server_access:
      key_pair_name: myapp_ci
      id_rsa_path: ~/.ssh/id_rsa
    github_private_ssh_key_path: ~/.ssh/id_rsa

For security, the lobot:install task added config/ci.yml to the .gitignore file since it includes sensitive AWS credentials and your CI users password.
Keep in mind that the default build script ci_build.sh uses the headless gem and jasmine. You'll want to add those to your Gemfile or change your build command.

At this point you will need to create a commit of the files generated or modified and push those changes to your remote git repository so jenkins can execute your build command.

## Modify the soloistrc if necessary

Switch postgres to mysql, or add your own recipes for your own dependencies.

## Starting your lobot instance

1. Launch an instance, allocate and associates an elastic IP and updates ci.yml:

    rake ci:server_start

2. Bootstrap the instance using the boostrap_server.sh script generated by Lobot. The script instals ruby prerequisites, creates the app_user account, and installs RVM for that user:

    cap ci bootstrap

3. Upload the contents of your chef/cookbooks/ directory, upload the soloistrc, and run chef:

    cap ci chef

Your lobot instance should now be up and running. You will be able to access your CI server at: http://<your instance address>/ with the username and password you chose during configuration.
For more information about Jenkins CI, see http://jenkins-ci.org/

## Troubleshooting

Shell access for your instance

    rake ci:ssh

Terminating your instance and deallocating the elastic IP

    rake ci:terminate

Suspending your instance

    rake ci:stop

## Add your new CI instance to [cimonitor](http://github.com/pivotal/cimonitor) and CCMenu

Lobot can generate the config for you, just run:

    rake ci:info

## Dependencies

* fog
* capistrano
* capistrano-ext
* rvm (the gem - it configures capistrano to use RVM on the server)
* nokogiri

# Tests

Lobot is tested using rspec, generator_spec and cucumber.  Cucumber provides a full integration test which can generate a rails application, push it to github, start a server and bring up CI for the generated project.
You'll need a git repository (which should not have any code you care about) and an AWS account to run the whole test suite.  It costs about $0.50 and takes about half an hour.
It will attempt to terminate the server however you should verify this so you do not get charged additional money.
Use the secrets.yml.example to create a secrets.yml file with your account information.

# Contributing

Lobot is in its infancy and we welcome pull requests.  Pull requests should have test coverage for quick consideration.

# License

Lobot is MIT Licensed and © Pivotal Labs.  See LICENSE.txt for details.
