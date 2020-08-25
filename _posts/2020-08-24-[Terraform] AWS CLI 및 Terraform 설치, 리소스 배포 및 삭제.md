## 1. Terraform 기본
Terraform은 HashiCorp 사의 코드형 인프라스트럭쳐 도구이다.  
Terraform은 클라우드, 물리적 시스템, VM 등 인프라의 자동화된 배포를 제공한다.  

</br>

## 2. AWS IAM 구성
- IAM 사용자 추가
  - 사용자
    - 사용자 이름: terraform
    - AWS 액세스 유형 선택: 프로그래밍 방식 액세스
  - 그룹
     - 그룹 이름: Terraform-administrators
     - 정책 이름: Administrator Access
- 액세스 키 ID 및 비밀 액세스 키가 담긴 CSV 파일 저장

</br>

## 3. AWS CLI Terraform 설치
- AWS CLI 설치    
`sudo apt install awscli`
- AWS CLI 구성  
`aws configure`  
AWS Access Key ID  
AWS Secret Access Key  
Default region name  
Default output format  
- AWS CLI 설치 및 구성 확인  
`aws s3 ls`
- Terraform 설치  
`wget https://releases.hashicorp.com/terraform/0.13.0/terraform_0.13.0_linux_amd64.zip`  
`unzip terraform_0.13.0_linux_amd64.zip`  
`sudo mv terraform /usr/local/bin/terraform`  
- Terraform 설치 확인  
`terraform`
`terraform -v`

</br>

## 4. 리소스 배포 및 삭제
Terraform으로 인프라를 정의하는 데 사용하는 파일을 Terraform 구성 파일이라고 한다.   
Terraform 구성 파일은 자체 디렉토리에 있어야 한다.  
- 디렉토리 생성  
`mkdir aws-example`
- 디렉토리 변경  
`cd aws-example`
- 구성 파일 작성  
`vi example.tf`

```tf
	provider "aws" {
	  profile = "default"
	  region = "ap-northeast-2"
	}
	
	resource "aws_instance" "example" {
	  ami = "ami-05438a9ce08100b25"
	  instance_type = "t2.micro"
	}
```
- provider 블록: 관리할 인프라 공급자를 지정하는 데 사용 
  - profile 속성: 자격 증명 프로파일
  - region: 인프라 공급자의 관리할 리전 선택
- resource 블록: 리소스 정의
  - 리소스 유형: aws_instance
  - 리소스 이름: example
  - ami: AWS의 AMI 이미지 ID
  - instance_type: AWS의 인스턴스 타입

- 초기화  
`terraform init`  
- 구성 파일의 유효성 검증  
`terraform validate`  
- 변경 사항 계획  
`terraform plan`  
- 변경 사항 적용(자동으로 terraform plan 실행)  
`terraform apply`  
- 현재 상태 검사  
`terraform show`  
- 인스턴스 삭제   
`terraform destroy`  

![Screenshot_from_2020-08-24_14-51-51](https://user-images.githubusercontent.com/53208493/91113237-075f0b00-e6c0-11ea-9b3c-643fe708ca53.png)
![Screenshot_from_2020-08-24_15-14-45](https://user-images.githubusercontent.com/53208493/91113239-0928ce80-e6c0-11ea-9f88-f835c8252a02.png)

</br>

## 5. 리소스 종속성
`vi example.tf`

```bash
# 인스턴스 두 개 정의 
provider "aws" {
  profile = "default"
  region = "ap-northeast-2"
}

resource "aws_instance" "example1" {
  ami = "ami-05438a9ce08100b25"
  instance_type = "t2.micro"
  
  depends_on = [aws_s3_bucket.example]
}

resource "aws_instance" "example2" {
  ami = "ami-05438a9ce08100b25"
  instance_type = "t2.micro"
}

# 하나의 인스턴스에 Elastic IP 적용
resource "aws_eip" "ip" {
  vpc = true
  instance = aws_instance.example1.id
}

# S3 버킷 적용
# S3 버킷의 이름은 AWS에서 유일해야 함
resource "aws_s3_bucket" "example" {
  bucket = "minju-test-bucket"
  acl = "private"
}
```

위 구성 파일에 정의한 리소스 및 속성의 주요 속성  
- aws_instance: AWS 인스턴스  
	- ami: 이미지 ID
	- instance_type: 인스턴스 타입
- aws_eip: Elastic IP  
	- vpc: EIP가 VPC에 있는지 여부
	- instance: EIP를 할당할 EC2 인스턴스의 ID
- aws_s3_bucket: S3 버킷  
	- bucket: S3 버킷 이름
	- acl: 버킷의 접근 제어
	
- 변경 사항 적용  
`terraform apply`

### 1) 암시적 종속성
암시적 종속성은 Terraform이 자동으로 리소스 간에 의존성을 분석해 리소스의 생성/변경/삭제 등 순서를 결정한다.    
example1 인스턴스와 ip 리소스가 "암시적 종속성"에 해당된다.    
```
resource "aws_eip" "ip" {
  vpc = true
  instance = aws_instance.example1.id
}
```
Elastic IP를 할당하기 전에 인스턴스가 생성되어 있어야 한다.  

### 2) 명시적 종속성  
명시적 종속성은 리소스 정의 시 사용자가 직접 리소스 간에 의존성을 명확하게 정의하는 것을 의미한다.  
example1 인스턴스는 example S3 버팃을 의존하고 있다.  
```
resource "aws_instance" "example1" {
  ami = "ami-05438a9ce08100b25"
  instance_type = "t2.micro"
  
  depends_on = [aws_s3_bucket.example]
}

resource "aws_s3_bucket" "example" {
  bucket = "minju-test-bucket"
  acl = "private"
}
```

</br>

## 6. 프로비저너
### 1) 프로비저너
프로비저너(Provisioner)는 Terraform 자체에서 제공하는 구성 관리 기능이다.  
Terraform의 내장 프로비저너  
- chef: Chef 구성 관리 도구  
- file: 파일 및 디렉토리 복사  
- habitat: Chef Habitat  
- local-exec: Terraform이 실행되는 로컬에서 실행  
- puppet: Puppet 구성 관리 도구  
- remote-exec: 원격 인스턴스에서 실행  
- salt-masterless: SaltStack 구성 관리 도구  

### 2) 프로비저너를 이용한 구성 파일 리소스 정의
다음은 remote-exec 및 local-exec를 정의한 Terraform 구성 파일이다.
```tf
provider "aws" {
  profile = "default"
  region = "ap-northeast-2"
}
resource "aws_key_pair" "example" {
  key_name = "example-key"
  public_key = file("./mykey.pub")
}
resource "aws_instance" "example" {
  ami = "ami-0d63024969e6f5191"
  instance_type = "t2.micro"
  key_name = aws_key_pair.example.key_name
  connection {
    type = "ssh"
    user = "ubuntu"
    private_key = file("./mykey")
    host = self.public_ip
  }
  provisioner "remote-exec" {
    inline =  [
      "sudo apt update",
      "sudo apt install nginx",
      "sudo systemctl enable nginx",
      "sudo systemctl start nginx"
    ]
  }
  provisioner "local-exec" {
    command = "echo ${aws_instance.example.public_ip} > ipaddr.txt"
  }
}
```
- remote-exec 프로비저너: connection 블록의 속성을 이용하여 접속해, 원격 인스턴스에서 실행되는 스크립트 정의  
- local-exec 프로비저너: Terraform이 설치된 로컬 시스템에서 실행되는 스크립트 정의  
 
### 3) 변경 사항 적용
- 인스턴스에 사용할 SSH 키 쌍 생성  
`ssh-keygen -f mykey`
- 변경 사항 적용  
`terraform apply`

인스턴스 생성 시 보안 그룹을 지정하지 않으면 default 보안 그룹이 사용된다.    
하지만 이 default 보안 그룹은 외부에서 SSH 접속이 허용되지 않는다.  

### 4) 구성 파일 수정
```tf
provider "aws" {
  profile = "default"
  region = "ap-northeast-2"
}
resource "aws_key_pair" "example" {
  key_name = "example-key"
  public_key = file("./mykey.pub")
}
resource "aws_instance" "example" {
  ami = "ami-0d63024969e6f5191"
  instance_type = "t2.micro"
  key_name = aws_key_pair.example.key_name
  vpc_security_group_ids = [aws_security_group.example-sg.id] // 보안 그룹 추가
  connection {
    type = "ssh"
    user = "ubuntu"
    private_key = file("./mykey")
    host = self.public_ip
  }
  provisioner "remote-exec" {
    inline =  [
      "sudo apt update",
      "sudo apt install nginx",
      "sudo systemctl enable nginx",
      "sudo systemctl start nginx"
    ]
  }
  provisioner "local-exec" {
    command = "echo ${aws_instance.example.public_ip} > ipaddr.txt"
  }
}

resource "aws_security_group" "example-sg" {
  name = "allow-ssh-web"
  egress {
    from_port = 0
    to_port = 0
    protocol = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
  ingress {
    from_port = 22
    to_port = 22
    protocol = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  ingress {
    from_port = 80
    to_port = 80
    protocol = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```
보안 그룹에서 인바인드 규칙은 SSH와 HTTP를 열어주고, 아웃바운드 규칙은 모든 포트를 열어준다.    

### 5) 변경 사항 적용
`terraform apply`  

### 6) Nginx 웹 서버 접속
![Screenshot from 2020-08-25 10-38-47](https://user-images.githubusercontent.com/53208493/91149002-7c006c80-e6f5-11ea-94a0-8e1615c0b8c0.png)
