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


## VPC/Public & Private Subnet/RTB/SG/IGW

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

## VPC/Subnet/RTB/SG/IGW/EC2/NLB

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
  MyVPC:
    Type: AWS::EC2::VPC
    Properties:
     CidrBlock: 10.0.0.0/16
     Tags:
        - Key: Name
          Value: MyVPC

  CustomVPC:
    Type: AWS::EC2::VPC
    Properties:
     CidrBlock: 20.0.0.0/16
     Tags:
        - Key: Name
          Value: CustomVPC

  MyIGW:
    Type: AWS::EC2::InternetGateway
    Properties:
      Tags:
        - Key: Name
          Value: My-IGW

  CustomIGW:
    Type: AWS::EC2::InternetGateway
    Properties:
      Tags:
        - Key: Name
          Value: Custom-IGW

  MyIGWAttachment:
    Type: AWS::EC2::VPCGatewayAttachment
    Properties:
      InternetGatewayId: !Ref MyIGW
      VpcId: !Ref MyVPC

  CustomIGWAttachment:
    Type: AWS::EC2::VPCGatewayAttachment
    Properties:
      InternetGatewayId: !Ref CustomIGW
      VpcId: !Ref CustomVPC

  MyPublicRT:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId: !Ref MyVPC
      Tags:
        - Key: Name
          Value: My-Public-RT

  MyDefaultPublicRoute:
    Type: AWS::EC2::Route
    DependsOn: MyIGWAttachment
    Properties:
      RouteTableId: !Ref MyPublicRT
      DestinationCidrBlock: 0.0.0.0/0
      GatewayId: !Ref MyIGW

  CustomPublicRT:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId: !Ref CustomVPC
      Tags:
        - Key: Name
          Value: Custom-Public-RT

  CustomDefaultPublicRoute:
    Type: AWS::EC2::Route
    DependsOn: CustomIGWAttachment
    Properties:
      RouteTableId: !Ref CustomPublicRT
      DestinationCidrBlock: 0.0.0.0/0
      GatewayId: !Ref CustomIGW

  MyPublicSN:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref MyVPC
      AvailabilityZone: !Select [ 0, !GetAZs '' ]
      CidrBlock: 10.0.0.0/24
      Tags:
        - Key: Name
          Value: My-Public-SN

  CustomPublicSN:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref CustomVPC
      AvailabilityZone: !Select [ 0, !GetAZs '' ]
      CidrBlock: 20.0.0.0/24
      Tags:
        - Key: Name
          Value: Custom-Public-SN

  MyPublicSNRouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      RouteTableId: !Ref MyPublicRT
      SubnetId: !Ref MyPublicSN

  CustomPublicSNRouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      RouteTableId: !Ref CustomPublicRT
      SubnetId: !Ref CustomPublicSN

  WebSG:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: WebSG
      VpcId: !Ref MyVPC
      GroupName: WebSG
      Tags:
        - Key: Name
          Value: WebSG
      SecurityGroupIngress:
      - IpProtocol: tcp
        FromPort: '80'
        ToPort: '80'
        CidrIp: 0.0.0.0/0
      - IpProtocol: tcp
        FromPort: '22'
        ToPort: '22'
        CidrIp: 0.0.0.0/0

  CustomWebSG:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: CustomSG
      VpcId: !Ref CustomVPC
      GroupName: CustomSG
      Tags:
        - Key: Name
          Value: Custom-WebSG
      SecurityGroupIngress:
      - IpProtocol: tcp
        FromPort: '80'
        ToPort: '80'
        CidrIp: 0.0.0.0/0
      - IpProtocol: tcp
        FromPort: '22'
        ToPort: '22'
        CidrIp: 0.0.0.0/0

  MyEC2:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: t2.micro
      ImageId: !Ref LatestAmiId
      KeyName: !Ref KeyName
      Tags:
        - Key: Name
          Value: My-EC2
      NetworkInterfaces:
        - DeviceIndex: 0
          SubnetId: !Ref MyPublicSN
          GroupSet:
          - !Ref WebSG
          AssociatePublicIpAddress: true

  CustomWeb1EC2:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: t2.micro
      ImageId: !Ref LatestAmiId
      KeyName: !Ref KeyName
      Tags:
        - Key: Name
          Value: Custom-WEB-1
      NetworkInterfaces:
        - DeviceIndex: 0
          SubnetId: !Ref CustomPublicSN
          GroupSet:
          - !Ref CustomWebSG
          AssociatePublicIpAddress: true
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
            yum install -y httpd
            systemctl start httpd && systemctl enable httpd
            echo "<html><h1>Endpoint Service Lab - CloudNeta Web Server 1</h1></html>" > /var/www/html/index.html

  CustomWeb2EC2:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: t2.micro
      ImageId: !Ref LatestAmiId
      KeyName: !Ref KeyName
      Tags:
        - Key: Name
          Value: Custom-WEB-2
      NetworkInterfaces:
        - DeviceIndex: 0
          SubnetId: !Ref CustomPublicSN
          GroupSet:
          - !Ref CustomWebSG
          AssociatePublicIpAddress: true
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
            yum install -y httpd
            systemctl start httpd && systemctl enable httpd
            echo "<html><h1>Endpoint Service Lab - CloudNeta Web Server 2</h1></html>" > /var/www/html/index.html

  CustomNLBTG:
    Type: AWS::ElasticLoadBalancingV2::TargetGroup
    Properties:
      Name: Custom-NLB-TG
      Port: 80
      Protocol: TCP
      VpcId: !Ref CustomVPC
      Targets:
        - Id: !Ref CustomWeb1EC2
          Port: 80
        - Id: !Ref CustomWeb2EC2
          Port: 80

  CustomNLB:
    Type: AWS::ElasticLoadBalancingV2::LoadBalancer
    Properties:
      Type: network
      Scheme: internet-facing
      Subnets:
        - !Ref CustomPublicSN
      Tags:
        - Key: Name
          Value: Custom-NLB

  NLBListener:
    Type: AWS::ElasticLoadBalancingV2::Listener
    Properties:
      DefaultActions:
        - Type: forward
          TargetGroupArn: !Ref CustomNLBTG
      LoadBalancerArn: !Ref CustomNLB
      Port: 80
      Protocol: TCP
```