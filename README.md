# aosp-android-build-manual
AOSP 안드로이드 소스코드를 직접 빌드하여 픽셀 스마트폰에 플래싱 하는 방법을 가이드합니다

## 1. 소스코드 다운로드

우리가 연구에서 사용할 기기는 구글에서 레퍼런스 스마트폰으로 출시하고 있는 Pixel 시리즈 입니다

Pixel의 경우 실행 가능한 AOSP 코드가 오픈되어 있으며 이를 토대로 빌드 + 플래싱 하여 실행 가능합니다


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
    
    `repo init`를 실행하여 repo 디렉토리를 초기화 합니다.

    다음 링크에서 브랜치명을 확인합니다 [소스 코드 태그 및 빌드](https://source.android.com/docs/setup/about/build-numbers?hl=ko#source-code-tags-and-builds)를 참고하세요.

    예를 들어 Pixel 4 XL 의 경우 빌드가능한 브랜치 중 android-13.0.0_r31 라는 태그명을 가진 브랜치가 최신임 을 알 수 있습니다

    *마스터 이외의* 브랜치를 확인하려면 `-b` 옵션을 통해 지정합니다.

    pixel 4 XL의 경우 아래와 같이 초기화 합니다    
    
    ```
    repo init -u https://android.googlesource.com/platform/manifest -b android-13.0.0_r31
    
    ```
    
    
    
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

## 2. 소스코드 빌드


### 환경설정

다음과 같이 `envsetup.sh` 스크립트로 환경을 초기화합니다.

```
source build/envsetup.sh

```

또는

```
. build/envsetup.sh

```

### 타겟 선택(lunch명령어)

`lunch`를 사용하여 빌드할 타겟을 선택합니다. `lunch
 aosp_{코드네임}-userdebug` 라고 실행할 경우 해당 타겟의 userdebug 모드로 빌드가 됩니다.
 userdebug 모드는 디버깅, 

정확한 구성은 인수로 전달될 수 있습니다. 

```
lunch aosp_{코드네임}-userdebug

```
예를 들어, Pixel 4 XL의 경우 본 가이드의 pre-requisite에서 확인했던 타겟 코드네임 coral을 확인하고 아래와 같이 실행해줍니다.

```
lunch aosp_coral-userdebug

```

인수 없이 실행하면 `lunch` 실행 후에 메뉴에서 타겟을 선택하라는 메시지가 표시되지만 메뉴에 모든 선택 항목이 포함되어 있지는 않습니다. [기기 빌드 선택](https://source.android.com/docs/setup/build/running?hl=ko#selecting-device-build)을 참고하여 AOSP에서 지원하는 모든 기기의 빌드 구성을 알아보거나 작업 중인 기기의 올바른 lunch와 관련해 팀원과 논의하세요.

모든 빌드 타겟이 `BUILD-BUILDTYPE` 양식을 사용합니다. 여기에서 `BUILD`는 특정 기능 조합을 참조하는 코드명입니다. `BUILDTYPE`은 다음 중 하나입니다.

| Buildtype | 용도 |
| --- | --- |
| user | 액세스 제한, 프로덕션에 적합 |
| userdebug | user 빌드와 유사하지만 루트 액세스 및 디버그 기능이 있으며 디버깅에 적합 |
| eng | 디버깅 도구가 추가된 개발 구성 |

`userdebug` 빌드는 `user` 빌드와 동일하게 작동해야 하며 일반적으로 플랫폼의 보안 모델을 위반하는 추가 디버깅을 사용할 수 있습니다. 이로 인해 `userdebug` 빌드는 뛰어난 진단 기능으로 사용자 테스트에 적합합니다. `userdebug` 빌드를 사용하여 개발할 때는 [userdebug 가이드라인](https://source.android.com/docs/setup/create/new-device?hl=ko#userdebug-guidelines)을 따르세요.

`eng` 빌드는 플랫폼에서 작업하는 엔지니어의 엔지니어링 생산성을 우선적으로 적용합니다. `eng` 빌드는 탁월한 사용자 환경을 제공하는 데 사용되는 다양한 최적화 기능을 사용 중지합니다. 그러지 않으면 `eng` 빌드는 `user` 및 `userdebug` 빌드와 유사하게 동작하므로 기기 개발자가 이러한 환경에서 코드가 어떻게 동작하는지 확인할 수 있습니다.

현재 lunch 설정을 보려면 다음 명령어를 실행합니다.

```
echo "$TARGET_PRODUCT-$TARGET_BUILD_VARIANT"

```

실제 하드웨어에서의 빌드 및 실행에 관한 자세한 내용은 [기기 플래싱](https://source.android.com/docs/setup/build/running?hl=ko)을 참고하세요.


### 코드 빌드

이 섹션은 설정이 완료되었는지 확인하기 위한 간단한 요약입니다.

`m`으로 모든 것을 빌드하세요. `m`은 `-jN` 인수로 병렬 작업을 처리할 수 있습니다. `-j` 인수를 제공하지 않으면 빌드 시스템에서 시스템에 가장 적합한 병렬 작업의 수를 자동으로 선택합니다.

```
m

```

위에서 설명한 것처럼 `m` 명령줄에 이름을 나열하여 전체 기기 이미지 대신 특정 모듈을 빌드할 수 있습니다. 또한 `m`은 특수 목적을 위해 일부 pseudotarget을 제공합니다. 다음은 관련 예입니다.

- **`droid`** - `m droid`는 일반 빌드입니다. 기본 타겟에 이름이 필요하기 때문에 이 타겟이 여기에 있습니다.
- **`all`** - `m all`은 `m droid`가 하는 모든 것과 `droid` 태그가 없는 모든 것을 빌드합니다. 빌드 서버는 이를 실행하여 이 트리에 있고 `Android.mk` 파일이 있는 모든 것이 빌드되는지 확인합니다.
- **`m`** - 트리 맨 위에서 빌드를 실행합니다. 이 기능은 하위 디렉터리에서 `make`을 실행할 수 있기 때문에 유용합니다. `TOP` 환경 변수 집합이 있으면 이 변수를 사용합니다. 그렇지 않으면 트리의 맨 위를 찾으려고 현재 디렉터리에서 트리를 조회합니다. 인수 없이 `m`를 실행하여 소스 코드 트리 전체를 빌드하거나 이름을 지정하여 특정 타겟을 빌드할 수 있습니다.
- **`mma`** - 현재 디렉터리의 모든 모듈과 종속 항목을 빌드합니다.
- **`mmma`** - 제공된 디렉터리의 모든 모듈과 종속 항목을 빌드합니다.
- **`croot`** - 트리의 상단으로 이동(`cd`)합니다.
- **`clean`** - `m clean`은 이 구성의 모든 출력 파일과 중간 파일을 삭제합니다. 이는 `rm -rf out/`과 동일한 기능을 합니다.

`m`이 제공하는 다른 pseudotarget을 보려면 `m help`를 실행합니다.

## 3. 플래시

빌드가 완료되면 out/target/product/{타겟별 브랜치} 디렉토리로 들어가면 system.img 이미지를 포함해 빌드 산출물 파일이 위치해 있습니다

### 디바이스 언락
직접 빌드한 이미지로 부팅하려면 언락이 필요합니다
다음 과정대로 디바이스를 언락합니다 [언락 매뉴얼](https://palpit.tistory.com/entry/%EA%B5%AC%EA%B8%80-%ED%94%BD%EC%85%80-4-XL-%EC%96%B8%EB%9D%BD%ED%95%98%EA%B8%B0Google-Pixel-4-XL-Unlock)

### 부트로더 진입
Pixel 기종의 경우 전원 버튼과 볼륨 + 버튼을 함께 15초간 누르면 부트로더 모드로 부팅됩니다

### fastboot 명령어 설치
 ```
sudo apt get install fastboot
```

### 이미지 플래싱
 out/target/product/{타겟별 브랜치} 디렉토리에 있는 산출이미지를 아래와 같이 플래싱합니다
 ```
fastboot --disable-verity --disable-verification flash vbmeta vbmeta.img
fastboot reboot fastboot
fastboot flash product product.img
fastboot flash system system.img
fastboot flash system_ext system_ext.img
fastboot reboot
```
