## 문제 상황
로컬 sts workspace에 문제가 생겨 해당 워크스페이스에서 작업한 코드들이 마구 뒤섞인 상태였다..!   
하지만 다행히 github 저장소에 코드들을 올려놓았었고, 그 코드들을 다시 sts로 불러와 작업을 해보려고 한다.

## 해결 방법
1. Git Repository 연동
- Git Repository 창에서 **Clone a Git repository** 선택
- 자신의 Git repository의 URI를 복사한 후 아래 URI 칸에 붙여넣는다.
<img width="344" alt="1" src="https://user-images.githubusercontent.com/53208493/129362908-90f8e70a-5c93-4083-8a9f-d8dd991f8c5e.PNG">  

- Git과 연동된 Project를 저장할 Local Directory Path를 적어준다.
<img width="343" alt="3" src="https://user-images.githubusercontent.com/53208493/129363007-28b8e1f3-685b-4faa-9960-13fb9d6f8159.PNG">

2. Maven Project Import
- Clone 했던 Git Repository를 오른쪽 마우스로 클릭한 후 Import Projects를 선택한다.
<img width="483" alt="1" src="https://user-images.githubusercontent.com/53208493/129363525-b490ce75-9c8d-401c-9d99-b87cc236ec9d.PNG">

- import할 프로젝트를 선택한 후 Finish 버튼을 클릭하면 STS가 알아서 import를 진행해준다.
- 만약 Spring이 제대로 동작하지 않는다면 Maven -> Update Project를 해주자.
