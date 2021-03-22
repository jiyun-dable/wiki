# Dable AWS 관련 설정
---

## AWS CLI setting 방법

`brew install aws`
`brew install awscli`

## AWS secret 설정 방법

(2021-03-22 기준) 팀장님(정현님)께 Secret key를 Authy 앱으로 발급 받으세요.

## AWS configure setting 방법

```bash
AWS Access Key ID [None]:
AWS Secret Access Key [None]:
Default region name [None]: ap-northeast-2
Default output format [None]: # 비워도 됩니다.
```

## AWS secret 파일 저장 방법 알기

`~/.aws`에서 confidential 파일에서 secret key와 ID를 관리하세요.

## AWS Region 설정하기
`~/.aws`에서 config 파일에서 region을 관리하세요.

## AWS 사용 방법

`aws ec2 help`
`aws s3 ls`

## 주의사항

Secret key는 절대 공개하지 마세요.
