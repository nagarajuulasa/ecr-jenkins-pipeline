node {
  stackname = "Nginx-ECS"
  buildnumber = env.BUILD_NUMBER
  ecrRegistry = "https://[Account Number].dkr.ecr.us-east-1.amazonaws.com"
  stage 'Checkout'
    checkout scm

  stage 'Docker  configuration'
    sh 'aws ecr get-login --region us-east-1 | xargs xargs'

  stage 'Build New Docker Image'
    docker.withRegistry("${ecrRegistry}") {
      docker.build('mynginx').push(env.BUILD_NUMBER)
    }

  stage 'Update Service in Stack'
    sh "aws cloudformation update-stack --stack-name ${stackname} --use-previous-template --capabilities CAPABILITY_NAMED_IAM  --region us-east-1  \
    --parameters ParameterKey=VPC,UsePreviousValue=true ParameterKey=Subnets,UsePreviousValue=true ParameterKey=KeyName,UsePreviousValue=true \
    ParameterKey=TemplateBucket,UsePreviousValue=true ParameterKey=Repository,UsePreviousValue=true ParameterKey=RepositoryTag,ParameterValue=${buildnumber}"

  stage 'Wait for Completion'
    result = sh(returnStdout: true, script: "aws cloudformation describe-stacks --stack-name ${stackname} --region us-east-1 --query 'Stacks[*].StackStatus' --output text")
    for (int i = 0; i < 1000; i++) {
      result = sh(returnStdout: true, script: "aws cloudformation describe-stacks --stack-name ${stackname} --region us-east-1 --query 'Stacks[*].StackStatus' --output text")
      if (result.contains("ERROR") || result.contains("ROLLBACK")) {
         error: "Error in Stack Build"
      } else if (result.contains('COMPLETE')) {
        break;
      }
      echo "Status ${result}"
      sleep: "10s"
    }
}
