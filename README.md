# Satellite API

[![CircleCI](https://circleci.com/gh/sat-utils/sat-api.svg?style=svg)](https://circleci.com/gh/sat-utils/sat-api)

*One API to search public Satellites metadata*

# sat-api

v1.0: This API uses Elastic Search as its engine and uses on AWS's Lambda and APIGateway to update and serve the data. A live API of the master branch is deployed to https://sat-api.developmentseed.org

v0.3.0 (sat-api legacy): A sat-api legacy version can be found on the [legacy branch ](https://github.com/sat-utils/sat-api/tree/legacy) and is deployed at https://api.developmentseed.org/satellites.

Documentation is available at http://docs.sat-utils.org/ and can be contributed to [here](https://github.com/sat-utils/sat-api-express/).

## Deployment

First, make sure you have AWS credentials with necessary access to AWS Lambda and AWS APIGateway (an admin level access will do enough):

- You MUST create a bucket on S3 that is used for storing deployment artifacts and metadata csv files.
- Update `.kes/config.yml` and enter the name of the bucket.
- If direct access to the elasticsearch instance is needed from fixed IP address, copy `.env.example` to `.env` and add your IP address there.
-  In the APIGateway console select the sat-api, click on the Binary Support menu on the left and add `'*'` as the Binary media type.

    $ kes cf deploy -r us-east-1
    
Replace us-east-1 with any desired region. This will deploy the CloudFormation stack, which includes API Gateway, Lambda functions, Step Functions, CloudWatch Rules, Elasticsearch, and associated IAM Roles. Additional calls of 'kes cf deploy' will update the existing CloudFormation stack.

The Landsat and Sentinel ingestors are run as Step Functions every 12 hours (Landsat at 6:00 and 18:00, Sentinel at 0:00 and 12:00), as can be seen under the CloudWatch Rules console. They can be disabled from the console.

## Elasticsearch Management

A Lambda function is included that provides Elasticsearch management functions from within the AWS Lambda Console. Navigate to the Lambda functions console for the region the sat-api stack has been deployed to and locate the *stackName*-manager Lambda function. From here you can configure test events that consist of a JSON payload.

```
{
    "action": action,
    "index": "sat-api"
}
```

Unless it has been changed in the code, the main index used in the Elasticsearch instance will always be sat-api. The action parameter can be:

- putMapping: Puts a new mapping for indexing. This is done automatically during ingestion if it does not already exist so should never need to be used directly.
- deleteIndex: This deletes the index. Use with caution!
- listIndices: This returns a list of all indices. Unless some have been added should include 'sat-api' and '.kibana'
- reindex: Spawns a reindexing of all records

## Development

A live API of the develop branch is auto-deployed to https://sat-api-dev.developmentseed.org

To further develop the API, install dependenceis with yarn and build the files for deployment (using webpack). This creates moduels under dist/ that can be deployed as the Lambda function source code.

    $ yarn install
    $ yarn build

    # to continually watch and build source files
    $ yarn run watch

## About
[sat-api](http://github.com/sat-utils/sat-api.git) was made by [Development Seed](http://developmentseed.org).
