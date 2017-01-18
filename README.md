# alexa
[![Build Status](https://travis-ci.org/frankgreco/alexa.svg?branch=master)](https://travis-ci.org/frankgreco/alexa)

Welcome to all of my Alexa skills! For now, i'm hosting them all in the same repository for ease of reference. However, this will most likely change in the future as the published skills scale.

[AWS Lambda](https://aws.amazon.com/lambda/) is being used to host the source code for all of the skills.

See each skill's README for more information.

## directory structure
This project has a top level `skills/` directory that contains a list of directories each corresponding to a different Alexa skill. Each skill contains a directory that contains all of the speech assets associated with that skill and another directory that contains the source code. Here's an example structure:

```sh
├── README.md
├── deploy.sh
└── skills
    ├── skillOne
    │   ├── assets
    │   │   ├── customSlotTypes
    │   │   │   ├── CUSTOM_TYPE
    │   │   ├── intentSchema.json
    │   │   └── sampleUtterances.txt
    │   └── src
    │       ├── index.js
    │       └── package.json
    └── skillTwo
        ├── assets
        │   ├── customSlotTypes
        │   │   ├── CUSTOM_TYPE
        │   ├── intentSchema.json
        │   └── sampleUtterances.txt
        └── src
            ├── index.js
            └── package.json
```

## continuous deployment
This project contains a `deploy.sh` file which will update and re-deploy the source code for each Alexa skill through our build tool, [Travis-CI](https://travis-ci.org/).

***NOTE:*** due to current limitations of Alexa, no cli tools or apis exist for updating or creating skills although it is on Amazons roadmap to do so. See [this post](https://forums.developer.amazon.com/questions/42864/automate-alexa-deployments.html) for more info.

***NOTE:*** the `deploy.sh` file assumes you already have an AWS Lambda function already created for each skill. Later, I hope to make it so that if a function doesn't already exist, it will just create it for you.

***NOTE:*** in each skills `src/package.json` file, the `name` paramater **must** match the name of your AWS lambda function.

The `deploy.sh` script loops through each skill in the `skills/` directory and performs the following steps to complete a successful deployment for each skill:

1. set an environment variable `FUNCTION_NAME` to the name parameter of the `package.json` file associated with that skill.
2. set an environment variable `ARN` with the Amazon resource name (arn) that uniquely identifies your AWS Lambda function. The format of an arn is `arn:aws:lambda:$AWS_DEFAULT_REGION:$ACCOUNT_NUMBER:function:$FUNCTION_NAME`. The `AWS_DEFAULT_REGION` and `ACCOUNT_NUMBER` environment variables is set in the `.travis.yml` file. If you would like to encrypt these variables, see [here](https://docs.travis-ci.com/user/environment-variables/).
3. install all of the dependencies for your skill (if any). **Important:** If your `index.js` file `requires` any code from within your projects (e.g. `require('./dependency')`), it must be included as a node_module! The easiest way to do this is to just create `src/dependency/` and include it as a dependency in your `src/package.json`: `"dependency": "file:dependency"`. When an `npm install` is performed, it will be included in the node_modules directory.
4. zip up `index.js` and `node_modules`
5. use the [awscli](https://aws.amazon.com/cli/) to upload the zip file to AWS Lambda and deploy the new code. **Important:** this command will only run successfully if you have the awscli configured for your account. In addition, you must have the `lambda:UpdateFunctionCode` permission in order to successfully update the code. While there are multiple ways to [configure](http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html) the awscli, I utilize environment variables and after encrypting them, set them in the `.travis.yml` file. Here are the environment variables that need to be set:
    * `AWS_ACCESS_KEY_ID`
    * `AWS_SECRET_ACCESS_KEY`
    * `AWS_DEFAULT_REGION` - this must be the **same** region that your AWS Lambda function is deployed in.

Again, as the published skills in this project scale, they will most likely be broken up into separate repositories so stay tune.
