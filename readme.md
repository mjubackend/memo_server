# 실행환경

memo.py 는 Python3 Flask 로 되어있습니다.

**본 과제는 최종적으로 AWS 에 서비스가 동작해야 됩니다. 실습 서버에서 개발 후 AWS 로 옮겨 설치해도 되고, AWS 에서 바로 작업을 해도 무방합니다.**

# 필요 패키지 설치

필요한 패키지 목록은 `requirements.txt` 에 있습니다. `pip` 을 이용해 필요 패키지를 설치합니다.
(필요시 `virtualenv` 환경을 이용하세요.)

```
$ pip install -r requirements.txt
```

# 실행 예시

일반적인 flask 실행 방식대로 실행하면 됩니다. 다만 bind 하는 주소를 0.0.0.0 으로 하기 위해서 `--host` 옵션을 추가합니다.

```
$ flask --app memo run --port 포트 번호 --host 0.0.0.0
```
또는
```
$ python3 memo.py
```

후자의 방법으로 실행할 경우 memo.py 안에서 port 번호 8000번을 기본값으로 사용하고 있으니 이 부분은 수정하세요.

# 동작 설명

## index.html 읽어 오기

`memo.py` 를 실행하고 브라우저에서 `http://mjubackend.duckdns.org:본인포트번호` 처럼 접근할 경우 `index.html` 을 읽어오게 됩니다.

이는 `Flask` 의 `template` 기능을 사용하고 있으며, 사용되고 있는 `index.html` 의 template file 은 `templates/index.html` 에 위치하고 있습니다.

이 template 은 현재 `name` 이라는 변수만을 외부 변수 값으로 입력 받습니다. 해당 변수는 유저가 현재 로그인 중인지를 알려주는 용도로 사용되며 `index.html` 은 그 값의 유무에 따라 다른 내용을 보여줍니다.

## index.html 이 호출하는 REST API 들

`index.html` 은 `memo.py` 에 다음 API 들을 호출합니다.

* `GET /login` : authorization code 를 얻어오는 URL 로 redirect 시켜줄 것을 요청합니다. (아래 [네이버 로그인 API 호출](#네이버-로그인-API-호출) 설명 참고)

* `GET /memo` : 현재 로그인한 유저가 작성한 메모 목록을 JSON 으로 얻어옵니다. 결과 JSON 은 다음과 같은 형태가 되어야 합니다.
  ```
  {"memos": ["메모내용1", "메모내용2", ...]}
  ```

* `POST /memo` : 새 메모를 추가합니다. HTTP 요청은 다음과 같은 JSON 을 전송해야 됩니다.
  ```
  {"text": "메모내용"}
  ```
  새 메모가 생성된 경우 memo.py 는 `200 OK` 를 반환합니다.


## 네이버 로그인 API 호출

수업 시간에 설명한대로 authorization code 를 얻어오는 동작은 클라이언트에서부터 시작하게 됩니다.

그런데 코드를 보면 `index.html` 에서 해당 API 동작을 바로 시작하는 것이 아니라 `GET /login` 을 통해서 서버에게 해당 REST API 로 redirect 시켜달라고 하는 방식으로 브라우저가 API 를 호출합니다. 이는 Chrome 계열의 브라우저의 `CORS` 문제 때문에 그렇습니다.

비록 서버가 redirect 해주는 방식을 사용하고는 있지만, 클라이언트인 브라우저가 그 API 를 직접 호출한다는 점은 동일합니다.

네이버 로그인을 사용하기 위해서는 수업 시간에 설명한 대로 [네이버 개발자 센터](https://developers.naver.com) 에 애플리케이션을 만들고 로그인 API 를 추가해야됩니다.

거기서 얻어진 client id, client secret, redirect uri 를 `memo.py` 의 소스 코드 안에 복사해 놔야 됩니다.

## 로그인 혹은 가입 처리

네이버 OAuth 과정을 마무리 한 뒤에 네이버의 profile API 를 통해 얻은 유저의 고유 식별 번호를 갖는 유저가 DB 에 없는 경우 새 유저로 취급하고 DB 에 해당 유저의 이름을 포함하는 레코드를 생성합니다.

만일 같은 네이버 고유 식별 번호의 유저가 있다면 그냥 로그인 된 것으로 간주합니다.

어떤 경우든 DB 에서 해당 유저의 정보를 얻어낼 수 있도록 `userId` 라는 `HTTP cookie` 를 설정합니다.


# 과제 목표

`memo.py` 의 내용을 채워서 메모장이 동작하게 구현하고 이를 AWS 에 Application load balancer 를 이용해 서비스를 구성하는 것이 과제의 목표입니다.

# Part1: memo.py 구현하기

## 1) `def home()`

`userId` 쿠키가 설정되어 있는 경우 DB 에서 해당 유저의 이름을 읽어와서 `index.html` template 에 반영하는 동작이 누락되어 있습니다.

## 2) `def onOAuthAuthorizationCodeRedirected()`

현재 `def onOAuthAuthorizationCodeRedirected()` 의 내용은 비어있습니다. 해당 함수에 코멘트로 표시된 대로 단계 1 ~ 4 까지를 채워 넣어야 합니다.

## 3) `def getMemos()`

메모 목록을 DB 로부터 읽어오는 부분이 빠져있습니다. 현재 로그인한 유저의 메모들을 읽어올 수 있어야 합니다.

## 4) `def post_new_memo()`

새 메모를 DB 에 저장하는 부분이 빠져있습니다. 현재 로그인한 유저의 메모로 저장되어야 합니다.


# Part2: DB 사용

DB 는 본인이 원하는 DB 중 어떤 것이라도 쓸 수 있습니다. (예: MySQL, MariaDB, Redis, MongoDB) DB 를 하나만 써도 되고 필요하다면 여러 DB 를 사용해도 무방합니다.

단 DB 설치는 docker 를 통해서 이루어져야 합니다. Docker 에 대한 설명은 수업 시간 강의 내용을 참고해주세요.

## 1) 실습 서버에서 DB 컨테이너 띄우기

수업 시간에 docker 에 대해서 배운대로 DB 를 띄우면 됩니다. 다른 학생과 포트가 겹치지 않도록 `-p` 옵션을 이용해 50000 + 실습번호 형태로 사용하기 바랍니다.

## 2) AWS 에 서비스를 구성할 때 DB 띄우기

DB 용 Ubuntu 가상 서버를 하나 띄우고, 그 가상 서버에서 docker 를 이용해 동일한 DB 를 띄웁니다.
가상 서버를 만든 후 Docker 의 설치는 다음 명령을 이용합니다.
```
$ sudo apt-get install docker.io
```
그리고 docker 명령을 할 때는 `sudo` 를 앞에 붙여서 실행하세요. (예: `$ sudo docker container ps`)
원래 docker 는 관리자 권한이 있어야 되는데, 실습 서버에서는 교수가 여러분 아이디가 sudo 없이 docker 를 수행할 수 있게 설정했었습니다. 그때문에 AWS 에서 가상 서버를 만든 경우에는 `sudo` 를 앞에 붙여 줘야 합니다.

구현한 메모 서비스는 DB 서버에 접근할 때 DB 서버의 private IP 를 이용합니다. public IP 를 이용하거나 Elastic IP 를 부여해서 접근하는 경우는 감점 처리 되니 주의하세요.

# Part3: AWS 에 서비스 구성

## 1) 서비스 서버 구성

* 교수가 공유한 `mjubackend` AMI 를 이용해서 가상 서버를 만듭니다.
* 이 때 서비스 서버는 ssh 와 http 만 열려 있도록 security group 을 설정합니다.
* `mjubackend` AMI 이미지는 TCP 80 번 포트에 nginx 를 동작시키고 있습니다.
* nginx 설정에서는 `/memo` 라는 경로가 들어오면 `127.0.0.1:30001` 를 통해서 uwsgi 를 호출하게 되어있습니다.
* 127.0.0.1:30001 에는 uwsgi 가 동작하고 있습니다. 이 때 uwsgi 의 설정 파일은 홈 디렉터리 아래 있는 `uwsgi.ini` 파일입니다. (`$ netstat -an | grep 30001` 로 확인해보세요)
* 서비스로 돌고 있는 uwsgi 는 홈디렉터리 아래 있는 `/memo` 경로에 대해서 `memo.py` 를 호출합니다.
* 따라서 여러분이 작성한 `memo.py`, `references/`, `templates/` 을 복사해 오면 별다른 문제 없이 `http://서비스서버publicIP/memo` 접근 가능 여부를 확인해볼 수 있습니다.
* **(중요)**: 담당 교수가 서비스 서버에 로그인해볼 수 있도록 다음의 SSH public key 를 `authorized_keys` 파일에 추가합니다. ```ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIHyI345QnkwdhuOcV/AUTYbxKZ8u1ayqjzduSCsQ6jAd dkmoon@dkmoon-desktop```

## 2) DB 서버 만들기

* DB 용 Ubuntu 가상 서버를 만들고 docker 를 이용해 여러분이 사용한 DB 를 띄웁니다.
* 서비스 서버는 DB 서버에 private IP 를 통해서 접속하도록 설정합니다.
* **(중요)**: 담당 교수가 서비스 서버에 로그인해볼 수 있도록 다음의 SSH public key 를 `authorized_keys` 파일에 추가합니다. ```ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIHyI345QnkwdhuOcV/AUTYbxKZ8u1ayqjzduSCsQ6jAd dkmoon@dkmoon-desktop```
  
## 3) 서비스 서버 이미지 만들기

* 여러분이 작성한 `memo.py`, `references/`, `templates/` 을 복사한 뒤 가상 서버를 커스터마이징 했으면, 이 가상 서버를 이용해 AMI 이미지를 만들도록 합니다.
* 여기서 만들어진 이미지를 이용해 launch template 을 작성할 겁니다.

## 4) Launch template 작성

* 앞에서 만든 AMI 를 통해서 launch template 을 작성합니다.
* 사양은 `t2.micro` 로 하고, ssh 와 http 만 열려 있도록 security group 을 설정합니다.

## 5) Target group (서버팜) 생성

* 대상 그룹 (target group) 으로 server farm 을 만듭니다.

## 6) Application 로드 밸런서 생성

* 앞서 생성한 target group 을 위해서 Application 로드 밸런서를 만듧니다.

## 7) Auto scaling 설정

* Auto scaling group 을 만들어서 로드 밸런서가 최대 1대부터 최대 2대까지 서버를 자동 생성하게 설정합니다.

## 주의할 점

* `memo.py` 를 복사해올 때 redirect URI 나 DB 주소가 하드코딩 되어있을 경우 AWS 에 서비스를 구성할 때는 이들을 모두 수정해야 합니다.


# 평가 항목

* AWS 의 application load balancer 의 DNS 주소를 통해서 서비스가 잘 동작하는가?
- 네이버 OAuth 가 제대로 구현되었는가?
- `GET /memo` 가 제대로 구현되었는가?
-  `POST /memo` 가 제대로 구현되었는가?
-  지시한 대로 AWS 에 서비스가 구성되었는가?

# 제출물

본인 github repo 에 `memo_server` 라는 서브폴더를 만들어서 다음 파일들을 제출하세요.
* 완성된 `memo.py` 를 포함해서 수정 혹은 추가한 파일들
* 실행 방법 및 코드 설명을 담고 있는 `readme.md` 파일
* `AWS` 상에 동작하고 있는 배포된 실행 환경

# 실행 환경 채점 관련 (반복되는) 코멘트

* 교수는 여러분의 AWS 콘솔에 로그인할 수 있습니다.
* 그러나 콘솔에 로그인할 수 있는 것과 가상 서버에 로그인할 수 있는 것은 별개이니 위에 언급된 SSH public key 를 반드시 가상 서버의 `authorized_keys` 에 포함시켜서 교수가 ssh 로그인 할 수 있게 하세요.
* 만일 ssh 로그인을 못해서 채점을 못할 경우 불이익은 해당 학생이 책임져야 합니다.
