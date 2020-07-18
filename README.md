# multicloud_task2
Automate AWS cloud using TERRAFORM

### PROVIDER

![0](https://user-images.githubusercontent.com/45136716/87842354-a296e080-c8c9-11ea-919d-45945fe490f0.jpg)

***************************************************************************************************************
## TASK 2

1. Create Security group which allow the port 80.

2. Launch EC2 instance.

3. In this Ec2 instance use the existing key or provided key and security group which we have created in step 1.

4. Launch one Volume using the EFS service and attach it in your vpc, then mount that volume into /var/www/html

5. Developer have uploded the code into github repo also the repo has some images.

6. Copy the github repo code into /var/www/html

7. Create S3 bucket, and copy/deploy the images from github repo into the s3 bucket and change the permission to public readable.

8 Create a Cloudfront using s3 bucket(which contains images) and use the Cloudfront URL to  update in code in /var/www/html


![1](https://user-images.githubusercontent.com/45136716/87842355-a3c80d80-c8c9-11ea-8d03-15f75038a405.jpg)


********************************************************************************************************

![00](https://user-images.githubusercontent.com/45136716/87843006-69ad3a80-c8ce-11ea-8bdc-cd510242e986.png)


**AWS** is an amazon platform which provides open source public cloud computing services.

**Terraform** is an open source infrastructure as a code software tool created by Hashicorp.

### The main components involved in launching of a web server are:
1. Elastic Cloud Compute(EC2) is a part of Amazon’s cloudcomputing platform that allows users to rent virtual computers on which to run their own computer applications. It provides compute as a service to the users(CAAS).

2. Elastic File System(EFS) is a cloud storage service provided by (AWS) designed to provide scalable, elastic, concurrent with some restrictions and encrypted file storage for use with both AWS cloud

services and on-premises resources. In simple words, it provides File storage as a service(FSAAS).

3. CloudFront is a content delivery network (CDN) offered by Amazon Web Services . Content delivery networks provide a globallydistributed network of proxy servers which cache content, such as web videos or other bulky media, more locally to consumers, thus improving access speed for downloading the content.


## STEP1: Specifing Provider

**Provider** is used to specify the cloud provider that we are going to use as terraform has same syntex for all cloud platforms for which it downloads plugins.Here we are using AWS as provider.

```
#provider
provider "aws" {
  region = "ap-south-1"
  profile = "Soul"
}
```

## STEP2: Creating Security Group

This is security group, we are defining a firewall which has allowed SSH, HTTP & one more port through which EFS can communicate, the inbound or the traffic coming in is called ingress and the out bound or traffic going outside is called egress. CIDR defines the range.


```
resource "aws_security_group" "sc-1" {    
 name        = "sc-1"    
        description = "Allows SSH and HTTP"    
 vpc_id      = "vpc-98918cf0"      
 ingress {      
  description = "SSH"      
  from_port   = 22      
  to_port     = 22      
  protocol    = "tcp"      
  cidr_blocks = [ "0.0.0.0/0" ]    
 }       
 ingress {      
  description = "HTTP"      
  from_port   = 80      
  to_port     = 80      
  protocol    = "tcp"      
  cidr_blocks = [ "0.0.0.0/0" ]    
 }      
 egress {      
  from_port   = 0  
      to_port     = 0      
  protocol    = "-1"      
  cidr_blocks = ["0.0.0.0/0"]    
 }      
 tags = {      
  Name = "sc-1"    
 }  
}
```

![sc-1](https://user-images.githubusercontent.com/45136716/87842389-c35f3600-c8c9-11ea-8a65-ffb3bc003b23.png)

## STEP3: Launching EFS

This is to create EFS, this will create EFS cluster with the encryption done on the data in rest.


```
resource "aws_efs_file_system" "myefs"{   
 creation_token="my-efs"      
 tags = {  
  Name= "myefs"    
 }  
}   
resource "aws_efs_mount_target" "first" { 
  file_system_id = aws_efs_file_system.myefs.id  
 subnet_id = "subnet-efdde787"  
 security_groups= [aws_security_group.sc-1.id] 
}
```

![efs-2](https://user-images.githubusercontent.com/45136716/87842363-a9bdee80-c8c9-11ea-8b3a-8912b104b082.png)

## STEP4: Launching instance(EC2)

This will create the instance with some listed software and will also mount the EFS which we created.

```

resource "aws_instance" "lwos1" {    
 ami           = "ami-0732b62d310b80e97"      
 instance_type = "t2.micro"    
 key_name = "mainKey"   
 security_groups = [aws_security_group.sc1.id]    
 subnet_id = "subnet-efdde787"   
 associate_public_ip_address = "1"           
 connection {      
  type     = "ssh"      
  user     = "ec2-user"      
  private_key = file("D:/google downloads/myKeycc.pem") 
      host     = aws_instance.lwos1.public_ip    
 } 
 provisioner "remote-exec" {     
 inline = [        
 "sudo yum install httpd  php git -y",        
 "sudo systemctl restart httpd",        
 "sudo systemctl enable httpd",      
 ]    
 }      
 tags = {      
  Name = "lwos1"    
 }  
}

```

![instance 3](https://user-images.githubusercontent.com/45136716/87842366-ab87b200-c8c9-11ea-9137-55ffe79a62f6.png)


## STEP5: Creating S3 bucket


This will help us create S3 bucket, this works as a unified storage from where we will use cloud front to make it globally scaled using its power of doing CDN- Content Delevery Network.

```
resource "aws_s3_bucket" "bucket1" {    
 bucket = "bucket1"    
 acl    = "public-read"      
 versioning {      
  enabled = true  
   }   
 tags = {      
  Name = "bucket1"      
  Environment = "Dev"    
 }
}
```

![s3 bucket 4](https://user-images.githubusercontent.com/45136716/87842369-ad517580-c8c9-11ea-89f1-7c161e1d6238.png)


## STEP6: Uploading on S3 bucket


Uplaoding the the static data to the s3 bucket that we just created. Key is the name of the file after the object is uploaded in the bucket and source is the path of the file to be uploaded.

```
resource "aws_s3_bucket_object" "s3obj" {
 depends_on = [
    aws_s3_bucket.bucket1,
]
  bucket       = "bucket1"
  key          = "automatiiion.jpg"
  source       = "D:/google downloads/automatiiion.jpg"
  acl          = "public-read"
  content_type = "image or jpeg"
}
```

![s3 bucket 5](https://user-images.githubusercontent.com/45136716/87842387-c22e0900-c8c9-11ea-8bf4-97a437157f97.png)


## STEP7: Creating CloudFront


CloudFront is the service that is provided by the AWS in which they create small data centres where they store our data to achieve low latency. It will create a CloudFront distribution using an S3 bucket. In this bucket, we have stored all of the assets of our site like images, icons, etc.


```
resource "aws_cloudfront_distribution" "SoulCF" {      
 origin {          
  domain_name = "bucket1.s3.amazonaws.com"          
  origin_id = "S3-bucket1"              
  custom_origin_config {              
   http_port = 80              
   https_port = 80              
   origin_protocol_policy = "match-viewer"              
   origin_ssl_protocols = ["TLSv1", "TLSv1.1", "TLSv1.2"]          
  }      
 }               
 enabled = true    
 default_cache_behavior {          
  allowed_methods = ["DELETE", "GET", "HEAD", "OPTIONS", "PATCH", "POST", "PUT"]          
  cached_methods = ["GET", "HEAD"]          
  target_origin_id = "S3-bucket1"            
  forwarded_values {              
   query_string = false                        
   cookies {                 
    forward = "none"              
   }          
  }          
  viewer_protocol_policy = "allow-all"          
  min_ttl = 0          
  default_ttl = 3600          
  max_ttl = 86400      
 }         
 restrictions {          
  geo_restriction {                           
   restriction_type = "none"          
  }  
     }        
 viewer_certificate {          
 cloudfront_default_certificate = true        
 }  
}
```

![cfd6](https://user-images.githubusercontent.com/45136716/87842362-a9255800-c8c9-11ea-80c7-6e523b93801f.png)


## We have to initialize it so that it can download AWS provider plugin.

![2](https://user-images.githubusercontent.com/45136716/87842356-a4f93a80-c8c9-11ea-9e2e-f5bee08fbbec.png)


## Applying complete code gives:

![apply](https://user-images.githubusercontent.com/45136716/87842358-a6c2fe00-c8c9-11ea-9401-c69324a1d0ef.png)

## Code start to excute


![2nd last](https://user-images.githubusercontent.com/45136716/87842357-a591d100-c8c9-11ea-8b69-99797df80142.png)


![last](https://user-images.githubusercontent.com/45136716/87842368-ac204880-c8c9-11ea-9aee-5c088440f5a1.png)

## what we get:-

![in last](https://user-images.githubusercontent.com/45136716/87842364-aa568500-c8c9-11ea-8f91-47c518cc05be.png)











































































































