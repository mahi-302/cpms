pipeline{
  agent {label 'main'}
    stages{
       stage('hosting application'){
        steps{
          sh "ls"
          sh "aws rds create-db-instance --db-instance-identifier test-mysql1 --db-name cpms --db-instance-class db.t2.micro --vpc-security-group-ids sg-08625c34c68a22785 --engine mysql --engine-version 5.7 --db-parameter-group-name default.mysql5.7 --publicly-accessible  --master-username admin --master-user-password mahesh8842 --allocated-storage 10 --region us-east-1"
          sleep(300)
          script{
              def cmd = "aws rds describe-db-instances --db-instance-identifier test-mysql1 --region us-east-1"
              def output = sh(script: cmd,returnStdout: true)
              jsonitem = readJSON text: output
              println(jsonitem)
           }
           sh "sudo sed -i.bak 's/endpoint/${jsonitem['DBInstances'][0]['Endpoint']['Address']}/g' userdata.txt"
          script{
              def cmd = "aws elbv2 create-load-balancer --name my-load-balancer --subnets subnet-05fc269a2a63b2a2e subnet-035a2f3e7b5674f9b --security-groups sg-08625c34c68a22785 --region us-east-1 "
              def output = sh(script: cmd,returnStdout: true)
              jsonitem1 = readJSON text: output
              println(jsonitem1)
              sleep(100)
            }
          script{
              def cmd = "aws elbv2 create-target-group --name my-targets --protocol HTTP --port 80 --target-type instance --vpc-id vpc-07d7c1ed0305d2070 --region us-east-1"
              def output = sh(script: cmd,returnStdout: true)
              jsonitem2 = readJSON text: output
              println(jsonitem2)
              sleep(180)
               }
           sh "aws elbv2 create-listener --load-balancer-arn ${jsonitem1['LoadBalancers'][0]['LoadBalancerArn']} --protocol HTTP --port 80 --default-actions Type=forward,TargetGroupArn=${jsonitem2['TargetGroups'][0]['TargetGroupArn']} --region us-east-1"
           sh "aws autoscaling create-launch-configuration --launch-configuration-name my-lc-cli --image-id ami-052efd3df9dad4825 --instance-type t2.micro --security-groups sg-08625c34c68a22785 --key-name awsall --iam-instance-profile ec2_full --user-data file://userdata.txt --region us-east-1"
           sh "aws autoscaling create-auto-scaling-group --auto-scaling-group-name my-asg-cli --launch-configuration-name my-lc-cli --max-size 2 --min-size 1 --desired-capacity 1 --target-group-arns ${jsonitem2['TargetGroups'][0]['TargetGroupArn']} --availability-zones us-east-1a --region us-east-1b"
        }
       }
  }
}
