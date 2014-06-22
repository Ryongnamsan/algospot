
# 알고스팟 개발 시작하기

알고스팟은 수많은 패키지들과 서버들이 합쳐져 돌아가고 있다. 따라서 이들을 각각 개인 서버에 셋업해 개발하는 것보다는, 가상머신을 이용하고 가상머신을 쉽게 셋업해 줄 수 있는 도구를 이용하기로 하자. 알고스팟의 공식 개발 환경은 [vagrant](http://vagrantup.com)를 이용한다. vagrant는 VirtualBox, VMWare 등의 가상화 도구를 커맨드 라인에서 사용하기 쉽게 포장해 둔 것이다.

## 필요한 도구들

* [Vagrant](http://vagrantup.com): 사용중인 운영체제 용을 다운받는다.
* [VirtualBox](https://www.virtualbox.org/): 가상 머신을 돌리기 위한 소프트웨어. VMWare, 리눅스의 경우 LXC를 이용해도 되지만 아직 확인해 보지 못했다.

## 체크아웃에서 개발 서버 돌리기까지


1. 먼저 git repository 를 클론하고, 외부 의존 리포지토리를 가져온다.

	$ git clone git://github.com/jongman/algospot.git  
	$ cd algospot  
	$ git submodule update --init --recursive

1. 가상 머신을 띄운다. 이 명령어는 가상 머신 이미지를 다운받고, 필요한 패키지를 깔고, 데이터베이스와 기타 서버들을 셋업해 준다. (테터링 중에는 쓰지말자)

	$ vagrant up && vagrant reload

1. 웹브라우저에서 http://localhost:8080/ 을 연다. 새 사용자를 만들거나 (아직 확인은 안해봄) admin/admin으로 로그인할 수 있다.

## 개발하기

1. 코드를 변경한 이후에는 다음 명령어를 내려 웹서버에 코드를 리로드한다.

	$ make restart-uwsgi

1. 이것이 귀찮을 경우 다음 명령을 내려 개발 서버를 연다. 개발 서버는 코드가 변경될 시 자동 리로드되므로 개발이 간편해진다.

	$ make runserver

## 새 언어 바인딩 추가하기

1. 언어 바인딩은 www/judge/languages/언어명.py에 추가하면 된다.
1. 컴파일 언어의 경우 cpp.py, 인터프리팅 되는 언어의 경우 py.py 를 참고해 작성한다.
1. 가상 머신에 컴파일러를 설치하는 과정을 ansible/single_box.yml에 추가한다. 이 파일은 [앤시블](http://docs.ansible.com/) 플레이북으로, 서버를 자동으로 설정할 수 있도록 한다.
1. 새 언어 바인딩을 추가한 이후, 다음 명령을 내려 웹서버와 채점 대몬을 재시작한다.

	$ make restart-uwsgi && make restart-celeryd

## 문제 해결하기

1. 문제가 생길 경우 다음 명령을 내려 가상 머신에 ssh 로 접속한다.

	$ vagrant ssh

1. 다음 명령을 내리면 장고 모델과 객체들에 접근할 수 있는 쉘에 접근할 수 있다.

	$ make shell

1. 다음 명령을 내리면 가상 머신 내의 DB 쉘에 접근할 수 있다.

	$ make dbshell

## 커밋하기

프로젝트에 새 기능을 추가하거나 버그를 고치려면 다음과 같은 과정을 밟는다.

1. [프로젝트 홈페이지](https://github.com/jongman/algospot)의 오른쪽 위 Fork  버튼을 이용해서 프로젝트를 fork 한다. 그러면 algospot 프로젝트의 개인 복사본이 생긴다.
1. 위 과정을 거쳐 해당 프로젝트를 클론한다.
1. 로컬에서 적절히 수정한 뒤 개인 복사본 프로젝트로 클론한다.
1. 개인 프로젝트 홈페이지에서 Pull Request 를 통해 jongman 에게 Pull Request 를 보낸다.
1. jongman 이 해당 프로젝트를 가져와 머지한다.
1. -끗-

좀더 자세한 설명은 [github의 매뉴얼 페이지](http://help.github.com/send-pull-requests/)에서 볼 수 있다.

## Troubles

* error at /accounts/register/ ([Errno 111] Connection refused)  
  회원가입 시, 위와 같은 에러가 난다면 개발서버에 회원가입 인증메일 발송을 위한 SMTP 서버가 없기 때문이다. 아래와 같이 로컬에 임시 SMTP 서버를 띄우고, local_setting.py에 SMTP 서버 포트를 지정한다.
        ```
        $ python -m smtpd -n -c DebuggingServer localhost:1025
        ```

        // local_settings.py  
        ```
        settings.EMAIL_HOST = 'localhost' 
        settings.EMAIL_PORT = 1025
        ```

* "The SSH connection was unexpectedly closed by the remote end."
  Vagrant up 명령어에서 위와 같은 명령어가 나온다면 놀라지 말고 `vagrant reload`를 해주면 된다.
