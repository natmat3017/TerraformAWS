provider "aws" {
  region = "us-east-1"
}

# Create a VPC
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
}

# Create a subnet
resource "aws_subnet" "main" {
  vpc_id     = aws_vpc.main.id
  cidr_block = "10.0.1.0/24"
}

# Create a network group
resource "aws_security_group" "allow_ssh" {
  name        = "allow_ssh"
  description = "Allow SSH inbound traffic"
  vpc_id      = aws_vpc.main.id

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port       = 0
    to_port         = 0
    protocol        = "-1"
    cidr_blocks     = ["0.0.0.0/0"]
  }
}

# Create an S3 bucket
resource "aws_s3_bucket" "terraform_state" {
  bucket = "my-terraform-state-bucket"

  versioning {
    enabled = true
  }
}

# Create a DynamoDB table
resource "aws_dynamodb_table" "terraform_locks" {
  name           = "terraform-locks"
  billing_mode   = "PAY_PER_REQUEST"
  hash_key       = "LockID"

  attribute {
    name = "LockID"
    type = "S"
  }
}

# Create 5 EC2 instances
resource "aws_instance" "web" {
  count         = 5
  ami           = "ami-0cff7528ff583bf9a" # Amazon Linux 2 AMI (HVM), SSD Volume Type
  instance_type = "t2.micro"
  subnet_id     = aws_subnet.main.id

  vpc_security_group_ids = [
    aws_security_group.allow_ssh.id
  ]

  associate_public_ip_address = true

  tags = {
    Name = "Web Server ${count.index + 1}"
  }
}
