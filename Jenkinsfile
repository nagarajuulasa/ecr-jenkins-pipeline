node {
  stackname = "Nginx-ECS"
  stage 'Checkout'
    checkout scm

  stage 'Docker  configuration'
    sh 'aws ecr get-login --region us-east-1 | xargs xargs'

  stage 'Build New Docker Image'
    docker.withRegistry('https://558201170204.dkr.ecr.us-east-1.amazonaws.com') {
      docker.build('mynginx').push(env.BUILD_NUMBER)
    }

  stage 'Update Service in Stack'
    sh 'aws cloudformation update-stack --stack-name ${stackname} --use-previous-template --capabilities CAPABILITY_NAMED_IAM  --region us-east-1 --profile isengard --parameters ParameterKey=VPC,UsePreviousValue=true ParameterKey=Subnets,UsePreviousValue=true ParameterKey=KeyName,UsePreviousValue=true ParameterKey=TemplateBucket,UsePreviousValue=true ParameterKey=Repository,UsePreviousValue=true ParameterKey=RepositoryTag,ParameterValue=latest'

  stage 'Wait for Completion'
    result = sh(returnStdout: true, script: "aws cloudformation describe-stacks --stack-name Nginx-ECS --region us-east-1 --query 'Stacks[*].StackStatus' --output text")
    for (int i = 0; i < 1000; i++) {
      result = sh(returnStdout: true, script: 'aws cloudformation describe-stacks --stack-name Nginx-ECS --region us-east-1 --query "Stacks[*].StackStatus" --output text')
      if (result.contains("ERROR") || result.contains("ROLLBACK")) {
         error: "Error in Stack Build"
      } else if (result.contains('COMPLETE')) {
        break;
      }
      echo "Status ${result}"
      sleep: 10s
    }
}
