## Git Branch Strategy란?
깃 브랜치를 효과적으로 관리하기 위한 워크플로우이다. </br>
워크플로우에는 git-flow, gitlab-flow, github-flow 등등 다양한 전략들이 있다. </br>

## Git Branch Strategy는 무엇이 제일 좋은가?
깃 브랜치 전략은 무엇이 제일 좋은가는 없다. 다만 Product에 적합한 전략이 있다. </br>
특정 브랜치에 머지가 되면 CI/CD툴을 활용해 코드가 테스트되고 바로 배포 되기때문에 Product 형태와 테스트 환경에 밀접하게 관련있다. </br>
그래서 Product의 배포 형태에 적합한 깃 브랜치 전략을 세우는 것이 중요하다. </br>
예를 들어 여러 버전을 사용하는 Product의 경우와 단일 버전을 사용하는 Product와는 깃 브랜치 전략은 다를 수 밖에 없을것이다. </br>

## 사용하고 있는 Git Branch Strategy

참고를 위해 내가 회사에서 서버 개발자로 일하면서 사용하는 깃 브랜치 전략을 공유하면 아래와 같다.

1. Master브랜치를 Feature브랜치로 Checkout한다.
1. Feature브랜치에서 작업을 한다.
1. Feature브랜치를 Dev에 Merge하고 Dev에서 Conflict를 수정하고 커밋을한다. 
1. Dev브랜치에서 테스트한다.
1. Feature브랜치를 Staging브랜치에 Merge하고 Staging에서 Conflict를 수정하고 커밋한다.
1. Staging에서 테스트한다.
1. Master브랜치를 Feature브랜치에 Merge하고 Feature에서 Conflict를 수정한다.
1. Master가 Merge된 Feature브랜치를 Master에 다시 Merge한다.

7번을 진행하는 이유는 Feature브랜치에서 코드를 좀더 깔끔하게 수정하고 Master에 머지하기 위해서 이렇게 진행한다.

이렇게하면 Dev -> Staging -> Product 전략에 비해서 장점이 하나 있다. </br>
만약 Staging에서 QA팀의 버그가 발생하면 Staging에서 다른 기능들도 Master로 배포를 못하기 때문이다. </br>
그러면 Cherry Pick을 통해서 특정 브랜치만 끄집에서 내서 배포해야하는 작업을 진행해야한다. </br>
이런 현상이나 Cherry Pick작업을 하지 않도록 위에서 공유한 브랜치 전략을 사용하고 있다. </br>
