# aosp-android-build-manual

## 1. 소스코드 다운로드

우리가 연구에서 사용할 기기는 구글에서 레퍼런스 스마트폰으로 출시하고 있는 Pixel폰 입니다

Pixel에서 실행 가능한 AOSP 코드가 오픈되어 있으며 이를 토대로 빌드 + 플래싱 하여 실행 가능합니다


### Pre-requisite: 

구글은 픽셀기기마다 고유한 코드네임을 가지고 있습니다. 다음링크에서 확인 가능합니다.  [기기 빌드 선택](https://source.android.com/docs/setup/build/running?hl=ko#selecting-device-build)을 참고하세요.

후술하겠지만 AOSP 10 버전 부터는 *repo* 명령어가 *python3*와 호환되니 *python3*를 사용하는 것을 추천드립니다


### Repo 명령어 설치

다음 링크의 Install 항목에 따라 *repo* 명령어를 설치 바랍니다 [Repo 명령어 설치](https://gerrit.googlesource.com/git-repo/+/refs/heads/master/README.md#install)



### 작업 디렉토리 초기화

1.  
    
    작업 파일을 보관할 빈 디렉터리를 만듭니다. 다음처럼 원하는 이름을 지정합니다.
    
    ```
    mkdir WORKING_DIRECTORYcd WORKING_DIRECTORY
    ```
    
2.  
    
    Git을 실명과 이메일 주소로 구성합니다. Gerrit 코드 검토 도구를 사용하려면 [등록된 Google 계정](https://www.google.com/accounts?hl=ko)과 연결된 이메일 주소가 필요합니다. 메시지를 수신할 수 있는 실제 주소인지 확인합니다. 여기서 제공하는 이름은 코드 제출에 대한 속성에 표시됩니다.
    
    ```
    git config --global user.name Your Namegit config --global user.email you@example.com
    ```
    
3.              
    
    `repo init`를 실행하여 최근 버그가 수정된 최신 버전의 Repo를 다운로드합니다. Android 소스에 포함된 다양한 저장소가 작업 디렉터리 내에 배치되는 위치를 지정하는 매니페스트의 URL을 지정해야 합니다.
    
    ```
    repo init -u https://android.googlesource.com/platform/manifest
    
    ```
    
    다음으로 마스터 분기를 확인합니다.
    
    ```
    repo init -u https://android.googlesource.com/platform/manifest -b master
    
    ```
    
    *마스터 이외의* 브랜치를 확인하려면 `-b`로 지정합니다. 분기 목록은 [소스 코드 태그 및 빌드](https://source.android.com/docs/setup/about/build-numbers?hl=ko#source-code-tags-and-builds)를 참고하세요.
    
    ***Python 2의 경우***
    
    **경고:** [Python 2 지원 종료](https://www.python.org/doc/sunset-python-2/)에 설명된 대로 Python 2 지원은 2020년 1월 1일에 종료되었습니다. 모든 주요 Linux 배포판에서 Python 2 패키지 지원이 중단됩니다. 모든 스크립트를 Python 3로 이전하는 것이 좋습니다.
    
    **참고:** AOSP는 Python 2 및 Python 3 패키지의 자체 사본과 함께 제공되므로 소스 트리에 포함된 버전(예: [SEPolicy](https://android.googlesource.com/platform/system/sepolicy/+/6e60287a1f73c4f792350f7c86dc7bb3e1d6d623/build/Android.bp))을 사용할 수 있습니다. Google에서는 Android 소스 트리의 모든 스크립트를 Python 3로 이전하므로 삽입된 Python 2 사본이 지원 중단될 수도 있습니다. 
    자세한 내용은 [Python 2 코드를 Python 3로 포팅](https://docs.python.org/3/howto/pyporting.html)과 [Python 2 코드 지원 종료 도움말](https://python3statement.org/practicalities/)을 참고하세요.
    
    ***Python3의 경우***
    
    '`/usr/bin/env 'python' no such file or directory`' 오류 메시지가 표시되면 다음 해결 방법 중 하나를 사용합니다.
    
    Ubuntu 20.04.2 LTS가 새로 설치(업그레이드 아님)된 Linux 버전이면 다음을 실행합니다.
    
    ```
    sudo ln -s /usr/bin/python3 /usr/bin/python
    ```
    
    Git 버전 2.19 이상을 사용하는 경우 `repo init`를 실행할 때 `--partial-clone`을 지정할 수 있습니다. 이를 통해 Git의 [부분 클론](https://crbug.com/git/2) 기능을 사용하여 모든 항목을 다운로드하지 않고 필요할 때 Git 객체만 다운로드할 수 있습니다. 부분 클론을 사용하면 많은 작업이 서버와 통신해야 하므로 지연 시간이 짧은 네트워크를 사용하는 개발자라면 다음을 사용하세요.
    
    ```
    repo init -u https://android.googlesource.com/platform/manifest -b master --partial-clone --clone-filter=blob:limit=10M
    
    ```
    
    Windows OS에만 적용: 심볼릭 링크를 만들 수 없다는 오류 메시지가 표시되어 `repo init`가 실패하면 GitHub [심볼릭 링크 문서](https://github.com/git-for-windows/git/wiki/Symbolic-Links)를 참고하여 만들거나 심볼릭 링크 지원을 사용 설정합니다. 관리자가 아닌 경우 [관리자가 아닌 사용자가 심볼릭 링크를 만들도록 허용](https://github.com/git-for-windows/git/wiki/Symbolic-Links#creating-symbolic-links) 섹션을 참고하세요.
    

초기화가 완료되면 작업 디렉터리에 저장소가 초기화되었다는 메시지가 표시됩니다. 이제 클라이언트 디렉터리에 매니페스트와 같은 파일이 보관되는 `.repo` 디렉터리가 포함됩니다.

### Android 소스 트리 다운로드

기본 매니페스트에 지정된 저장소에서 Android 소스 트리를 작업 디렉터리로 다운로드하려면 다음을 실행합니다.

```
repo sync
```

동기화 속도를 높이려면 `-c`(현재 분기) 및 `-jthreadcount` 플래그를 전달합니다.

```
repo sync -c -j8
```

Android 소스 파일은 작업 디렉터리의 프로젝트 이름 아래에 다운로드됩니다.

출력을 표시하지 않으려면 `-q`(quiet) 플래그를 전달합니다. 모든 옵션은 [Repo 명령어 참조](https://source.android.com/docs/setup/create/repo?hl=ko)를 참고하세요.
