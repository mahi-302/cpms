pipeline{
  agent {label 'main'}
    stages{
       stage('hosting application'){
        steps{
          sh "ls"
          sh "aws rds create-db-instance --db-instance-identifier test-mysql-instance --db-name cpms --db-instance-class db.t2.micro --vpc-security-group-ids sg-086c66f49f3a1b1cb --engine mysql --engine-version 5.7 --db-parameter-group-name default.mysql5.7 --publicly-accessible  --master-username admin --master-user-password Nareshkumar --allocated-storage 10 --region us-west-2"
          sleep(450)
          script{
              def cmd = "aws rds describe-db-instances --db-instance-identifier test-mysql-instance --region us-west-2"
              def output = sh(script: cmd,returnStdout: true)
              jsonitem = readJSON text: output
              println(jsonitem)
           }
           sh "sudo sed -i.bak 's/endpoint/${jsonitem['DBInstances'][0]['Endpoint']['Address']}/g' userdata.txt"
          script{
              def cmd = "aws elbv2 create-load-balancer --name my-load-balancer --subnets subnet-004d6b7c79e79f5a8 subnet-0b41cb4f9d9022011 --security-groups sg-086c66f49f3a1b1cb --region us-west-2 "
              def output = sh(script: cmd,returnStdout: true)
              jsonitem1 = readJSON text: output
              println(jsonitem1)
              sleep(100)
            }
          script{
              def cmd = "aws elbv2 create-target-group --name my-targets --protocol HTTP --port 80 --target-type instance --vpc-id vpc-025f049d00134320a --region us-west-2"
              def output = sh(script: cmd,returnStdout: true)
              jsonitem2 = readJSON text: output
              println(jsonitem2)
              sleep(180)
               }
           sh "aws elbv2 create-listener --load-balancer-arn ${jsonitem1['LoadBalancers'][0]['LoadBalancerArn']} --protocol HTTP --port 80 --default-actions Type=forward,TargetGroupArn=${jsonitem2['TargetGroups'][0]['TargetGroupArn']} --region us-west-2"
           sh "aws autoscaling create-launch-configuration --launch-configuration-name my-lc-cli --image-id ami-0d70546e43a941d70 --instance-type t2.micro --security-groups sg-086c66f49f3a1b1cb --key-name ubuntu --iam-instance-profile demo_full_access --user-data file://userdata.txt --region us-west-2"
           sh "aws autoscaling create-auto-scaling-group --auto-scaling-group-name my-asg-cli --launch-configuration-name my-lc-cli --max-size 2 --min-size 1 --desired-capacity 1 --target-group-arns ${jsonitem2['TargetGroups'][0]['TargetGroupArn']} --availability-zones us-west-2c --region us-west-2"
        }
       }
  }
}
