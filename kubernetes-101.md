## Note

EKS의 data plain만 이용하고 있음.
- cluster 관리
- kuberctl은 kubernetes 컴포넌트 관리

AP팀에서 생성한 cluster 중 하나로 **cluster-spot-live**임.

## 생성하는 방법

eksctl: The official CLI for Amazon EKS
- brew로 설치할 수 있음.
- 참고 : awseks 굉장히 불편함.

### Cluster 생성
- `eksctl create cluster` 사용하면 알아서 cluster에 필요한 권한, role 같은 것을
    만듦.
  - 이 명령어를 한 사람만 admin에 들어 있고, 나머지 분들은 추가로 넣으면 된다.
  - cluster 설정을 yaml로 customize 할 수도 있음.
### Nodegroup 생성
- `eksctl create nodegroup`
  - spot node group을 만들 수 있음.
- auto-scaling 설정
  - 빈 pod 같은게 있는지 알아본다든가.

### EB 앱을 EKS로 단계별 이관 작업


### Q&A
Pod 언제 Restart?
  - OOM(Out of memory)으로 kill 될 때
  - Restart 안되게 설정 가능?
    - Exponentional back off 같은 걸 하거나
    - Crashing 상태가 되면 restart 안되게 함.
