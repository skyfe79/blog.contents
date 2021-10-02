---
title: "CloudFormation EC2 기초 템플릿"
date: 2021-10-02T17:03:00Z
draft: false
tags: ["devops"]
---

출처: [따라하며 배우는 AWS 네트워크 입문](http://www.yes24.com/Product/Goods/93887402)

```yml
Parameters:
  KeyName:
    Description: Name of an existing EC2 KeyPair to enable SSH access to the instances. Linked to AWS Parameter
    Type: AWS::EC2::KeyPair::KeyName
    ConstraintDescription: must be the name of an existing EC2 KeyPair.
  LatestAmiId:
    Description: (DO NOT CHANGE)
    Type: 'AWS::SSM::Parameter::Value<AWS::EC2::Image::Id>'
    Default: '/aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2'
    AllowedValues:
      - /aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2

Resources:
  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: !Ref LatestAmiId
      InstanceType: t2.micro
      KeyName: !Ref KeyName
      Tags:
        - Key: Name
          Value: WebServer
      SecurityGroups:
        - !Ref MySG
      UserData:
        Fn::Base64:
          !Sub |
            #!/bin/bash
            yum install httpd -y
            systemctl start httpd && systemctl enable httpd
            echo "<h1>Test Web Server</h1>" > /var/www/html/index.html

  MySG:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Enable HTTP access via port 80 and SSH access via port 22
      SecurityGroupIngress:
      - IpProtocol: tcp
        FromPort: 80
        ToPort: 80
        CidrIp: 0.0.0.0/0
      - IpProtocol: tcp
        FromPort: 22
        ToPort: 22
        CidrIp: 0.0.0.0/0
```


