---
title: "AWS CloudFormation 모음"
date: 2021-10-02T17:03:00Z
draft: false
tags: ["devops"]
---

출처: [따라하며 배우는 AWS 네트워크 입문](http://www.yes24.com/Product/Goods/93887402)

## EC2 인스턴스 생성 및 보안그룹 설정

```yaml
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


## VPC/Public & Private Subnet/RTB/IGW

```yaml
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
  CloudNetaVPC:
    Type: AWS::EC2::VPC
    Properties:
     CidrBlock: 10.0.0.0/16
     Tags:
        - Key: Name
          Value: CloudNeta-VPC

  CloudNetaIGW:
    Type: AWS::EC2::InternetGateway
    Properties:
      Tags:
        - Key: Name
          Value: CloudNeta-IGW

  CloudNetaIGWAttachment:
    Type: AWS::EC2::VPCGatewayAttachment
    Properties:
      InternetGatewayId: !Ref CloudNetaIGW
      VpcId: !Ref CloudNetaVPC

  CloudNetaPublicRT:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId: !Ref CloudNetaVPC
      Tags:
        - Key: Name
          Value: CloudNeta-Public-RT

  DefaultPublicRoute:
    Type: AWS::EC2::Route
    DependsOn: CloudNetaIGWAttachment
    Properties:
      RouteTableId: !Ref CloudNetaPublicRT
      DestinationCidrBlock: 0.0.0.0/0
      GatewayId: !Ref CloudNetaIGW

  CloudNetaPrivateRT:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId: !Ref CloudNetaVPC
      Tags:
        - Key: Name
          Value: CloudNeta-Private-RT

  CloudNetaPublicSN:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref CloudNetaVPC
      AvailabilityZone: !Select [ 0, !GetAZs '' ]
      CidrBlock: 10.0.0.0/24
      Tags:
        - Key: Name
          Value: CloudNeta-Public-SN

  CloudNetaPrivateSN:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref CloudNetaVPC
      AvailabilityZone: !Select [ 2, !GetAZs '' ]
      CidrBlock: 10.0.1.0/24
      Tags:
        - Key: Name
          Value: CloudNeta-Private-SN

  CloudNetaPublicSNRouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      RouteTableId: !Ref CloudNetaPublicRT
      SubnetId: !Ref CloudNetaPublicSN

  CloudNetaPrivateSNRouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      RouteTableId: !Ref CloudNetaPrivateRT
      SubnetId: !Ref CloudNetaPrivateSN

  CloudNetaSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Enable HTTP access via port 80 and SSH access via port 22
      VpcId: !Ref CloudNetaVPC
      SecurityGroupIngress:
      - IpProtocol: tcp
        FromPort: '80'
        ToPort: '80'
        CidrIp: 0.0.0.0/0
      - IpProtocol: tcp
        FromPort: '22'
        ToPort: '22'
        CidrIp: 0.0.0.0/0

  CloudNetaPublicEC2:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: t2.micro
      ImageId: !Ref LatestAmiId
      KeyName: !Ref KeyName
      Tags:
        - Key: Name
          Value: CloudNeta-Public-EC2
      NetworkInterfaces:
        - DeviceIndex: 0
          SubnetId: !Ref CloudNetaPublicSN
          GroupSet:
          - !Ref CloudNetaSecurityGroup
          AssociatePublicIpAddress: true

  CloudNetaPrivateEC2:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: t2.micro
      ImageId: !Ref LatestAmiId
      KeyName: !Ref KeyName
      Tags:
        - Key: Name
          Value: CloudNeta-Private-EC2
      NetworkInterfaces:
        - DeviceIndex: 0
          SubnetId: !Ref CloudNetaPrivateSN
          GroupSet:
          - !Ref CloudNetaSecurityGroup
      UserData:
        Fn::Base64:
          !Sub |
            #!/bin/bash
            (
            echo "qwe123"
            echo "qwe123"
            ) | passwd --stdin root
            sed -i "s/^PasswordAuthentication no/PasswordAuthentication yes/g" /etc/ssh/sshd_config
            sed -i "s/^#PermitRootLogin yes/PermitRootLogin yes/g" /etc/ssh/sshd_config
            service sshd restart
```