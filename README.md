<img src="https://merico-build.s3-us-west-2.amazonaws.com/Merico-Build-logo.svg" height="30"/>

![build passing](https://badgen.net/badge/build/passing/4c1?icon=gitlab)
![license](https://badgen.net/github/license/merico-dev/build)
[![Discord](https://img.shields.io/discord/844603288082186240.svg?label=&logo=discord&logoColor=ffffff&color=7389D8&labelColor=6A7EC2)](https://discord.gg/83rDG6ydVZ)

[中文README](README.zh-CN.md)

[Merico Build](<https://merico.build/>) is a web app empowering open source developers, maintainers, and communities with metrics from Git, GitHub, and more. The application uses novel static analysis to more accurately measure the value and impact of code contribution. 

- For maintainers, Merico Build renders these metrics to track community growth and to identify rising star contributors. 
- For developers, Merico Build automatically showcases your contributions in an evolving and sharable profile. 

**Why Merico Build?**

1) Code contribution is analyzed with more perceptive static code analysis, including [ELOC](https://playground.merico.build/), an enhanced noise-proof metric drastically improving upon Lines of Code (LOC.) 
2) Multidimensional signals extracted directly from the code, accounting for volume, dependencies, redundancy, test coverage, comment coverage, and more. 
3) Contributions get translated into badges. Share badges and celebrate accomplishments with proof.

We invite you to join us to improve the transparency, health, and sustainability of open source. Merico began as [research](<https://www2.eecs.berkeley.edu/Pubs/TechRpts/2018/EECS-2018-174.pdf>) in UC Berkeley's computer science department, we are excited to welcome contributors from around the world to join us in advancing this technology to empower developers everywhere to build better!

**To use the app, go to https://merico.build/. To contribute, scroll down for instructions to set up for development.**

**If you have a feature request, feel free to create an issue in this repo or join us on [Discord](<https://discord.gg/83rDG6ydVZ>)!**

## Summary

* [**Requirements**](#requirements)
* [**Installation**](#installation)
    * [**Installing build-frontend**](#installing-build-frontend)
    * [**Installing build-backend**](#installing-build-backend)
* [**Configuration**](#configuration)
* [**Usage**](#usage)
* [**Contributing**](#contributing)
* [**License**](#license)

<br/>

## Requirements

Postgres 12.5
<https://www.postgresql.org/download/>

NodeJS 14.x
<https://nodejs.org/>

<br/>

## Installation

To get the entire Merico Build application running on your computer, you'll need to clone and configure two different repositories: `build-backend` and `build-frontend`.

<br/>

## Installing build-frontend

### Clone

Clone [the repo](<https://github.com/merico-dev/build-frontend>)
```
git clone https://github.com/merico-dev/build-frontend.git
```

### Install dependencies
```
npm i
```
<br/>


## Installing build-backend

### Clone

Clone [the repo](<https://github.com/merico-dev/build-backend>)
```
git clone https://github.com/merico-dev/build-backend.git
```

### Install dependencies
```
npm i
```

### Initialize the submodule.
```
git submodule init
git submodule update
```

### Install `common-backend` dependencies.
```
cd common-backend
npm i
```

### Create the config file

Rename `config/local.example.js` to `config/local.js`
<br/>
#### Set up the DB

Requirement: Postgres 12.5

1. Create the DB
```
psql postgres -c "CREATE DATABASE your_db_name"
psql postgres -c "CREATE SCHEMA your_db_name"
psql postgres -c "CREATE USER merico PASSWORD 'merico'"
psql postgres -c "ALTER USER merico WITH SUPERUSER"
psql postgres -c "GRANT ALL ON ALL TABLES IN SCHEMA your_db_name to merico"
psql postgres -c "GRANT ALL ON ALL TABLES IN SCHEMA public to merico"
psql postgres -c "GRANT ALL ON SCHEMA your_db_name to merico"
```

2. Rename or copy `/config/database.example.json` to `/config/database.json`

3. Update the `development` config object in  `database.json`:
```
{
  "development": {
    "user": "merico",
    "host": "localhost",
    "database": "your_db_name",
    "password": "merico",
    "port": 5432,
    "dialect": "postgres"
  },
  ...
}
```

4. Create the tables
```
psql -f db/schema-only.sql -d your_db_name -U merico
psql -f db/badge-types.sql -d your_db_name -U merico
psql -f db/refresh-views.sql -d your_db_name -U merico
```
<br/>

## Configuration

`build-backend` requires a good deal of configuration. Most of it is required to make it work. For this section you'll be editing `config/local.js`.
<br/>

Please note, the Analytics Engine, RabbitMQ, and Minio are all **shared resources** for our open source developers. As such, you may receive other contributors' RabbitMQ messages. We are working towards making the Analytics Engine open source, but in the meantime we are offering this shared service.

<br/>

### GitHub Auth

Merico Build uses GitHub for Oauth and for access to the the user's public repo data.

1. [Create a GitHub OAuth app](<https://docs.github.com/en/developers/apps/building-oauth-apps/creating-an-oauth-app>) with the following settings:

Property | Value
------------ | -------------
Application name | `your-github-username-merico`
Homepage URL | `http://localhost:1337/`
Authorization callback URL | `http://localhost:1337/auth/github/callback`

2. Once your app is created, on the Settings page for your GitHub app, `Generate a new client secret`.

3. Copy and paste your GitHub OAuth App `Client ID` and `Client Secret` to the `githubAuth` config object in `config/local.js`.
<br/>

### GitLab Auth

1. As a logged-in Gitlab user, go to Preferences > Applications.
2. Create a new application.
3. Name it whatever you like.
4. Check the `read_api` and `read_user` permissions.
5. Set the `Redirect URI` to `http://localhost:1337/auth/gitlab/callback`
6. Save the `Application Id` and `Secret` and paste them into the `gitlabAuth` config object in `config/local.js`

<br/>

### Analytics Engine

Merico Build uses GRPC to execute functions on the Analytics Engine (AE). Merico will soon be making AE open source. Keep your eye out for it!

Update `CA_GRPC_SERVER` config value in `config/local.js` to the following:

`'52.41.108.240:30006'`

<br/>

### Minio
Merico Build uses Minio to store the protobuf files created by the Analytics Engine. The `common-backend` submodule reads these protobuf files and enters the data in the Build DB.

Update the `minio` config object in `config/local.js` to the following:
```
minio: {
  endPoint: '52.41.108.240',
  port: 30003,
  useSSL: false,
  accessKey: 'GmXnuzA3a27uO8RU',
  secretKey: 'ahx57GhFwZhEBlNENkz0TgVn',
  bucketAE: 'merico-analysis'
},
```
<br/>

### RabbitMQ

Build uses RabbitMQ to receive messages from the Analytics Engine about report progress and completion.

Update the `RABBIT_MQ_URL` config value in `config/local.js` to the following:

`'amqp://rabbitmq:9UvTFggbQsGhNx4hrRdA24tx@52.41.108.240:30001/rabbitmq'`

<br/>

### AWS S3

You'll need an AWS account for this. If you don't already have one, [sign up for a free AWS account](<https://portal.aws.amazon.com/billing/signup?refid=em_127222&redirect_url=https%3A%2F%2Faws.amazon.com%2Fregistration-confirmation#/start>).

1. [Create a bucket](<https://www.google.com/search?q=aws+s3+create+bucket&oq=aws+&aqs=chrome.0.69i59l3j0i20i263j69i57j69i60l3.1480j1j4&sourceid=chrome&ie=UTF-8>). Use the following access settings:

<img src="https://s3-us-west-2.amazonaws.com/cdn.mericobuild.com/public_access.png" width="500" />

2. [Create a folder](<https://docs.aws.amazon.com/AmazonS3/latest/userguide/using-folders.html#create-folder>) called `badges` within the new bucket.
3. Use an existing IAM user, or create a new one, and give them at least the S3 permission to `putObject` (for testing purposes you can just give them full S3 access if you like). Copy their `Access Key` and `Secret Access Key`.
4. Update the `AWS_S3` settings in `config/local.js` with the newly created `bucket name`, `Access Key`, `Secret Access Key`, and `Region`.
<br/>

### AWS SES

*Merico Build can function without SES. It will fail to send emails, but the app can handle these errors. If you do want to get email sending functionality working, follow these steps:

You'll need an AWS account for this. If you don't already have one, [sign up for a free AWS account](<https://portal.aws.amazon.com/billing/signup?refid=em_127222&redirect_url=https%3A%2F%2Faws.amazon.com%2Fregistration-confirmation#/start>).

1. Go to the AWS Simple Email Service console.
2. Verify your email domain.
3. Once your domain is verified, click on `SMTP Settings`.
4. Click on `Create my SMTP Credentials`
5. Copy the `Access Key` and `Secret Access Key`
6. Paste them in the SES section of `config/local.js`
<br/>

### Encryption Key

Update the `ENCRYPTION_KEY` config value in `config/local.js` to a random string of your choosing. We use this for various encryption functions.
<br/><br/>

## Usage

### build-backend
```
npm run dev
```

### build-frontend
```
npm run dev
```

After you've started the frontend app and the code has compiled, you should be able to visit http://localhost:3000/ and see the Merico Build homepage.

Sign up with your GitHub account, authorize the app, and submit a repository for analysis.

Note: it's best to stick with smaller repositories for now. We do limit analysis to the latest 2,500 commits, but even still, a really large repo (e.g. Django) can take up to a day to process. Try to limit your analyses to repos with under 5,000 commits.

<br/>

### Screenshots

Project Profile             |  My Repositories
:-------------------------:|:-------------------------:
<img src="https://merico-build.s3-us-west-2.amazonaws.com/js-callgraph.png" /> | <img src="https://merico-build.s3-us-west-2.amazonaws.com/my_repos.png" />

<br/> 

## Contributing

We welcome and appreciate contributions. Feel free to find your own way and put up a pull request, or join us on [our discord](<https://discord.gg/83rDG6ydVZ>) and we can help. 

<br/>

## License

Build is [Apache 2.0 licensed](<https://github.com/merico-dev/build/blob/master/LICENSE>)

## Testing

This is an edit to test notifications related to submitting a PR. Please reject it.

