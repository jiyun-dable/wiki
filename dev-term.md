# dev-term.dable.io 사용 방법 
---

별도 계정 없이 `dable.id_rsa`를 사용해서 `ubuntu@dev-term.dable.io`에 접속합니다.

## 1. 맨 처음 접속할 때 설정할 사항들 

### 1.1 OpenSSH Authentication agent에 private key를 추가합니다.

`ssh-add dable.id_rsa`

### 1.2 권한 변경

        Ubuntu@dev-term.dable.io: Permission denied (publickey).

User가`dable.id_rsa` 파일을 Read, Write를 할 수 있도록 권한 변경을 해줍니다.

`chmod 600 dable.id_rsa`

## 2. dev-term.dable.io 접속

`ssh ubuntu@dev-term.dable.io`로 접속합니다. 

## 3. 개인별 작업 디렉토리 설정

`$HOME/user` 아래에 `mkdir jiyun`을 하시고 그 디렉토리에서 작업을 진행해주세요.

