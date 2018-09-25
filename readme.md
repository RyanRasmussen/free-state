# Free State

Stateless Wordpress to deploy on AWS Elastic Beanstalk. Based on [Bedrock](https://roots.io/bedrock/)

## How to use this repo

This repository is intended to provide a boilerplate configuration for a new Wordpress website. Basically, you clone this repo into a new project, develop your theme locally and then deploy the project to a production environment on Elastic Beanstalk.

### Prerequisites

1. Install PHP Composer globally: https://getcomposer.org/download/
2. Install mysql and a web server locally. There are lots of ways to do this, but on a Mac, using MAMP is by far the easiest way to do it: https://www.mamp.info/en/downloads/

### Clone this project

1. Create a new directory for your project: `mkdir <project-name>`
2. Navigate to that directory: `cd <project-name>`
3. Clone this repo into the working directory: `git clone https://github.com/annelewisllc/free-state ./`
4. Remove all existing git data from boilerplate: `rm -rf .git`
5. Create a new `.env` file from the template: `cp .env.example .env`
6. Install dependencies: `composer install`

### Configure the local development database

1. Lanuch MAMP and start the web and database servers.
2. By default, MAMP automatically launches a standard start page. Note the mysql connection credentials on this page.
3. Connect to the local database. Create a database for your project. This is probably best done through the interface in a tool like Sequel Pro or MAMP's built-in version of phpMyAdmin, but it can be done with a sql command as well: `CREATE DATABASE <DATABASENAME>;`
4. Update your `.env` file with local development database connection details

### Configure local Wordpress

1. Generate Wordpress keys on [roots.io](https://roots.io/salts.html) and copy them into the `.env` file.
2. Update .env file with local development home page URL. This will vary from environment to envrionment based on the configuration of your MAMP install. It will likely look something like `http://localhost:8888/project-name/web/`

### Configure AWS S3 to store the media library

1. Create an S3 bucket for your project: http://docs.aws.amazon.com/AmazonS3/latest/gsg/CreatingABucket.html
2. Create a dedicated IAM user for your project: http://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_create.html#id_users_create_console
3. When asked to apply a permissions policy to the new user, use the template below to give this new user access to only the bucket for this project.

```
{
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:ListAllMyBuckets",
      "Resource": "arn:aws:s3:::*"
    },
    {
      "Effect": "Allow",
      "Action": "s3:*",
      "Resource": [
        "arn:aws:s3:::<BUCKET-NAME>",
        "arn:aws:s3:::<BUCKET-NAME>/*"
      ]
    }
  ]
}
```

4. Update your `.env` file with the name of the new bucket as well as the access key ID and the secret access key for this new IAM user.

### Launch and configure Wordpress

1. If they're not running already, launch the web server and the mysql server.
2. Navigate to the project home page. This should start the Wordpress installer and configuration utility.

### Develop your theme locally

Themes for your Wordpress project live in the `web/app/themes` folder. Drop your base and child themes into that folder and commit them to version control.

### Commit your code to a new private GitHub repository

GitHub has a good guide for adding an existing project to a new repo: https://help.github.com/articles/adding-an-existing-project-to-github-using-the-command-line/

### Deploy to Elastic Beanstalk.

Push your latest version of the site to github.

Compress your repository to a .zip for deployment.
`$ git archive -v -o myapp.zip --format=zip HEAD`

Create a new Web Server Environment.

1. Predefined configuration: PHP (for WP sites)
2. Environment type: Single instance
3. Upload the zip of your project and name your project
4. Create an RDS DB Instance
    - Instance type: t2.small
    - Health reporting system type: Basic
    - Root volume type: Magnetic
    - Root colume size: 5 GB

Configure your Elastic Beanstalk app

1. Document root: /web
2. Pull new Wordpress keys from [roots.io](https://roots.io/salts.html) and put them in the configuration as new variables
3. Fill in your database variables `DB_HOST`,`DB_NAME`,`DB_PASSWORD`,`DB_PORT`,`DB_PREFIX`,`DB_USER`
4. S3 content integration:  `S3_BUCKET_NAME`,`DBI_AWS_ACCESS_KEY_ID`,`DBI_AWS_SECRET_ACCESS_KEY`
5. Variables for letsencrypt config:
    - `BASE_DOMAIN` (no characters, just base domain. Ex: hirono-site-prod.us-east-1.elasticbeanstalk.com) You will update this to the new domain once it launches
    - `CERTIFICATE_EMAIL` - your email address, or can put todd@annelewisllc.com
6. Supercache variables:
    - `CACHEPATH` - relative path to directory, probably `/var/app/current/web/app/cache/`
    - `WPCACHEHOME` - relative path to directory, probably  `/var/app/current/web/app/mu-plugins/wp-super-cache/`
    - `WPCACHE_WPCORE_URL` - `/wp/`
6. Wordpress variables
    - `WP_HOME` - the base URL with characters, Ex: http://hirono-site-prod.us-east-1.elasticbeanstalk.com
    - `WP_SITEURL` - `WP_HOME` pointing to the /wp directory, Ex: http://hirono-site-prod.us-east-1.elasticbeanstalk.com/wp
    - `WP_ENV` - Stage of development: `development`, `production`, `staging`


