# Highload

## VPC
`aws ec2 create-vpc --cidr-block 10.10.0.0/18 --no-amazon-provided-ipv6-cidr-block --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=kma-genesis},{Key=Lesson,Value=public-clouds}]' --query Vpc.VpcId --output text`  

After it you should save VPC-id -> $VPC

## Subnets
`aws ec2 create-subnet --vpc-id $VPC --availability-zone eu-central-1a --cidr-block 10.10.1.0/24`  
`aws ec2 create-subnet --vpc-id $VPC --availability-zone eu-central-1b --cidr-block 10.10.2.0/24`  
`aws ec2 create-subnet --vpc-id $VPC --availability-zone eu-central-1c --cidr-block 10.10.3.0/24`  

Creating subnets (I creaste them for eu-central-1 region because I use them, should be replaced for your region)  
After entering of the commands you should save subnets-ids -> $SUB1 $SUB2 $SUB3

## Gateway
`aws ec2 create-internet-gateway --query InternetGateway.InternetGatewayId --output text`  
After entering of this command you should save gatway-id -> $IGW  
`aws ec2 attach-internet-gateway --internet-gateway-id $IGW --vpc-id $VPC`  

## Security group
`aws ec2 create-security-group --group-name kma-genesis-sg --description "kma highload sh" --vpc-id $VPC`  
After entering of this command you should save security group id -> $SG

`aws ec2 authorize-security-group-ingress --group-id $SG --protocol tcp --port 22 --cidr  0.0.0.0/0`  
`aws ec2 authorize-security-group-ingress --group-id $SG --protocol tcp --port 80 --cidr  0.0.0.0/0`  
`aws ec2 authorize-security-group-ingress --group-id $SG --protocol tcp --port 443 --cidr  0.0.0.0/0`  

## Creating template
`aws ec2 create-launch-template --launch-template-name GenesisScalingTempl --version-description AutoScaling1 --launch-template-data {\"NetworkInterfaces\":[{\"DeviceIndex\":0,\"AssociatePublicIpAddress\":true,\"Groups\":[\"$SG\"],\"DeleteOnTermination\":true}],\"ImageId\":\"ami-0233214e13e500f77\",\"InstanceType\":\"t3.micro\",\"TagSpecifications\":[{\"ResourceType\":\"instance\",\"Tags\":[{\"Key\":\"Name\",\"Value\":\"KmaAutoscaling\"}]}],\"BlockDeviceMappings\":[{\"DeviceName\":\"/dev/sda1\",\"Ebs\":{\"VolumeSize\":15}}]} --region eu-central-1`   
Image ID depend on region. Region should be the same as you use.  
For Windows use \ before "

## Autoscaling
`aws autoscaling create-auto-scaling-group --auto-scaling-group-name GenesisScalingGroup --launch-template "LaunchTemplateName=GenesisScalingTempl" --min-size 1 --max-size 2 --desired-capacity 1 --vpc-zone-identifier "$SUB1,$SUB2,$SUB3" --availability-zones "eu-central-1a" "eu-central-1b" "eu-central-1c"`

## Load balancer
`aws elbv2 create-load-balancer --name kma-genesis-lb  --subnets $SUB1 $SUB2 $SUB3 --security-groups $SG`
