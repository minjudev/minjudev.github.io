## 1. Terraform 기본
Terraform은 HashiCorp 사의 코드형 인프라스트럭쳐 도구이다.  
Terraform은 클라우드, 물리적 시스템, VM 등 인프라의 자동화된 배포를 제공한다.  

## 2. AWS IAM 구성
- IAM 사용자 추가
  - 사용자
    - 사용자 이름: terraform
    - AWS 액세스 유형 선택: 프로그래밍 방식 액세스
  - 그룹
     - 그룹 이름: Terraform-administrators
     - 정책 이름: Administrator Access
- 액세스 키 ID 및 비밀 액세스 키가 담긴 CSV 파일 저장

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


terraform init
# 구성 파일의 유효성 검증
terraform validate
# 변경 사항 계획
terraform plan
# 변경 사항 적용(자동으로 terraform plan 실행)
terraform apply
terraform show
# 인스턴스 삭제 
terraform destroy
```

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2b643e45-feee-457a-a76a-49a97b43b7b2/Screenshot_from_2020-08-24_14-51-51.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2b643e45-feee-457a-a76a-49a97b43b7b2/Screenshot_from_2020-08-24_14-51-51.png)

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/344f42e1-52f0-40e8-b736-074079b64a0b/Screenshot_from_2020-08-24_15-14-45.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/344f42e1-52f0-40e8-b736-074079b64a0b/Screenshot_from_2020-08-24_15-14-45.png)

- terraform.tfstate: 상태 파일

    상태 파일이 없으면 아무 것도 관리할 수 없음

    상태라는 걸 terraform이 따로 관리를 하고 있다!

    상태 파일을 잘 관리하는 것은 아주 중요하다!

- VPC

    default vpc와 그에 맞는 subnet이 존재해야 함

## 5. 리소스 종속성

- /home/student/aws-example/example.tf

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

# 리소스 생성 순서에 대해서는 terraform engine이 알아서 처리함!
# terraform은 절차형 언어가 아님
```

```bash
terraform apply
```
