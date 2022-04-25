## 파일 크기에 따라서 어떻게 전송/수신하나
- 2021년 깃허브 레포가 2억개라고 한다. 이 데이터를 다 어떻게 전송하고 수신할까? 특히 레포의 크기가 매우 크면 어떻게 관리할 수 있을까 


### 어떻게 DB를 구성할까? 
- https://stackoverflow.com/questions/30451559/how-the-github-store-your-repository-files
- If you wanted to store Git repositories, what you'd want to do is store them on a filesystem (or a cluster thereof) and then have a pointer in your database to point to where the filesystem is located, then use a library like Rugged or JGit or Dulwich to read stuff from the Git repository.
- 위의 글은 정확히 github가 이런 식으로 운영한다는 것을 말하는 것은 아니지만, 우리가 이렇게 운영할 수 있을 듯. 
- .git 폴더를 서버의 파일시스템에 저장함. 
- 폴더의 pointer를 db에 저장함
- https://www.pankajtanwar.in/blog/how-does-github-store-millions-of-repo-and-billions-of-files
- 캐시도 쓰는데 깃허브에 캐시가 유용할까?? 우리 시스템은 유저 수가 많지 않고 한 파일(레포)만 계속 고칠테니 유용할 듯! 


### 서버 파일 시스템에 모든 데이터를 다 저장 vs DB에 다 넣기?
- object는 key-value 구조로 되어있다. key는 SHA1 해시값, value는 내용. 이걸 레디스같은 key-value 기반 DB에 넣는 것도 괜찮을 듯. 레디스는 인메모리니 힘들 것 같긴 하다. 
- 파일 시스템으로 해서 HDFS같은걸 써보는 것도 괜찮을 듯. 
- DB에 저장하는 것 보다 파일시스템을 이용하는게 더 빠르니 파일시스템이 나을 것 같다. 