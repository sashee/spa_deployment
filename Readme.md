#### SPA deployment CloudFormation template

Deploy to the ```us-east-1``` region, wait ~20 minutes.

##### Outputs

* ```URL```: the deployment URL
* ```Bucket```: the bucket where you can put the files

##### Try it out

You need:

* The aws cli configured
* jq

Steps:

* clone this repo

```
git clone https://github.com/sashee/spa_deployment.git
```

* clone the [react-redux-typescript-boilerplate](https://github.com/rokoroku/react-redux-typescript-boilerplate)

```
git clone https://github.com/rokoroku/react-redux-typescript-boilerplate.git
```

* build the SPA:

```
cd react-redux-typescript-boilerplate
npm ci
npm run build
cd ..
```

* deploy the template:

```
cd spa_deployment
aws --region us-east-1 cloudformation deploy --template-file cloudformation.yml --stack-name spa-deployment-test --capabilities CAPABILITY_IAM
SPA_TEST_BUCKET=$(aws --region us-east-1 cloudformation describe-stacks --stack-name spa-deployment-test | jq -r '.Stacks[0].Outputs[] | select(.OutputKey == "Bucket") | .OutputValue')
SPA_TEST_URL=$(aws --region us-east-1 cloudformation describe-stacks --stack-name spa-deployment-test | jq -r '.Stacks[0].Outputs[] | select(.OutputKey == "URL") | .OutputValue')
cd ..
```

* deploy the sample application:

```
aws s3 sync react-redux-typescript-boilerplate/build s3://$SPA_TEST_BUCKET --delete --exclude '*.map'
```

* browse the url in a browser:

```
echo $SPA_TEST_URL
```

##### Delete the stack

* clear the bucket:

```
aws s3 rm s3://$SPA_TEST_BUCKET --recursive
```

* delete the stack (you need to issue multiple deletes during multiple hours to delete everything):

```
aws --region us-east-1 cloudformation delete-stack --stack-name spa-deployment-test

aws --region us-east-1 cloudformation describe-stacks --stack-name spa-deployment-test
```
