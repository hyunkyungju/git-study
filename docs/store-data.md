## github같은 곳은 git 정보를 어디에 저장하나
- git: 로컬에서 버전 관리
- github:  git 저장소를 관리하는 클라우드 기반 호스팅 서비스
- 그렇다면 github는 로컬의 .git 폴더 정보를 어떻게, 어디에 저장하는가?

### How the GitHub store your repository files?
https://stackoverflow.com/questions/30451559/how-the-github-store-your-repository-files

- GitHub uses Git to store repositories, and accesses those repos from their Ruby application
- repository가 .git 폴더를 의미하는 것인듯. 
- 깃허브는 원래 Grit이라는 루비 라이브러리를 사용했음. 지금은 rugged를 사용함.
- There are Git reimplementations in other languages like JGit for Java 
- If you wanted to store Git repositories(.git 폴더), what you'd want to do is store them on a filesystem (or a cluster thereof) and then have a pointer in your database to point to where the filesystem is located, then use a library like Rugged or JGit or Dulwich to read stuff from the Git repository.
- .git 폴더를 서버의 파일시스템에 저장하라는 뜻인듯. 
- pointer를 db에 가지라는 것이 무슨 말인지 잘 이해가 안감
- .git 폴더를 가져오면 JGit 같은 라이브러리로 내용에 접근할 수 있는 듯 하다. 
- There are plugins for a lot of popular web frameworks for doing user file uploads and file management. 
- 사용자가 다운로드할 수 있는 파일만 다운로드할 수 있도록 적절한 사용 권한을 구축하는 것이 중요하다. 다양한 프레임워크 플러그인이 어떻게 작동하는지 살펴볼 필요가 있다.

### JGit
https://git-scm.com/book/ko/v2/%EB%B6%80%EB%A1%9D-B%3A-%EC%95%A0%ED%94%8C%EB%A6%AC%EC%BC%80%EC%9D%B4%EC%85%98%EC%97%90-Git-%EB%84%A3%EA%B8%B0-JGit

