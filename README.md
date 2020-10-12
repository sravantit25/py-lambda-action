# Deploy specific directories to specific lambdas

This workflow builds on top of the excellent: https://github.com/mariamrf/py-lambda-action. It lets you deploy a specific directories to specific lambdas. 

## Background

The GitHub action `mariamrf/py-lambda-action` does a good job of packaging our lambda and dependencies and deploying it to AWS. However, it assumes that there is exactly one lambda per repo. The way [Qxf2](https://qxf2.com/?utm_source=py-lambda-action&utm_medium=click&utm_campaign=From%20GitHub) have organized our lambdas is different. We have one repo and we place one directory per lambda inside the repo. This makes using this GitHub action difficult. 

## Use
Deploys everything in the specified directory within the repo as code to the Lambda function, and installs/zips/deploys the dependencies as a separate layer the function can then immediately use.

### Pre-requisites
In order for the Action to have access to the code, you must use the `actions/checkout@master` job before it. See the example below.

### Structure
- Lambda code should be structured normally/as Lambda would expect it.
- **Dependencies must be stored in a `requirements.txt`** or a similar file (provide the filename explicitly if that's the case).

### Environment variables
Stored as secrets or env vars, doesn't matter. But also please don't put your AWS keys outside Secrets.
- **AWS Credentials**  
    That includes the `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, etc. It's used by `awscli`, so the docs for that [can be found here](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-envvars.html).

### Inputs
- `lambda_layer_arn`  
    The ARN for the Lambda layer the dependencies should be pushed to **without the version** (every push is a new version).
- `lambda_function_name`  
    The Lambda function name. [From the AWS docs](https://docs.aws.amazon.com/cli/latest/reference/lambda/update-function-code.html), it can be any of the following:
    - Function name - `my-function`  
    - Function ARN - `arn:aws:lambda:us-west-2:123456789012:function:my-function`  
    - Partial ARN - `123456789012:function:my-function`
- `lambda_directory`
    The directory with the lambda code
- `requirements_txt`
    The name/path for the `requirements.txt` file. Defaults to `requirements.txt`.

__Implementation__
1. I added a `lambda_directory` input argument to `action.yml`. 
2. Then, in `entrypoint.sh`, in the method `publish_function_code()` method, I added a `cd "${INPUT_LAMBDA_DIRECTORY}"` line just before we zip up the code.
3. I tagged this as v1.0.2


### Example workflow
```yaml
name: deploy-dummy-lambda
on:
  push:
    branches:
      - lambda-deploy-action
    paths:
      - 'dummy_lambda/**'
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@master
    - name: Deploy code to Lambda
      uses: qxf2/py-lambda-action@v1.0.2
      with:
        lambda_directory: 'dummy_lambda'
        lambda_function_name: arn:aws:lambda:ap-south-1:285993504765:function:dummyLambda
        requirements_txt: 'dummy_lambda/requirements.txt'
      env:
        AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
        AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        AWS_DEFAULT_REGION: ${{ secrets.SKYPE_SENDER_REGION }} 
```
