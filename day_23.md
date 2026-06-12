# 📘 Day 23 — Ubuntu 서버 환경 구축 + Tomcat 배포 + GitHub 연동

## 0. 핵심 빠른 참조

| 구분 | 명령어 | 설명 |
|------|--------|------|
| 관리자 전환 | `sudo su -` | root 권한 (#으로 변경) |
| 파일 편집 | `sudo nano 파일경로` | 터미널 텍스트 편집기 |
| nano 저장 | `Ctrl + O` → `Enter` | 저장 |
| nano 종료 | `Ctrl + X` | 나가기 |
| 패키지 업데이트 | `sudo apt-get update` | 패키지 목록 갱신 |
| Java 설치 | `sudo apt-get install openjdk-21-jdk` | JDK 21 설치 |
| 파일 다운로드 | `sudo wget "URL"` | URL에서 파일 다운 |
| 압축 해제 | `sudo tar -xvf 파일명.tar.gz` | tar.gz 압축 해제 |
| 권한 부여 | `sudo chmod 755 폴더명` | 읽기/실행 권한 |
| 목록 확인 | `ls -al` | 파일 목록 + 권한 표시 |
| 폴더 이동 | `cd 폴더명` / `cd ..` | 이동 / 상위 이동 |
| 파일 복사 | `sudo cp -r 원본 대상` | 재귀 복사 |
| Tomcat 시작 | `sudo ./bin/startup.sh` | 톰캣 구동 |
| Tomcat 종료 | `sudo ./bin/shutdown.sh` | 톰캣 중지 |
| 재부팅 | `sudo reboot` | 시스템 재시작 |
| 터미널 전환 | `Ctrl + Alt + F3` | GUI → CLI 전환 |

---

## 1. Ubuntu 초기 설정

### VM 설정
```text
ISO 선택 → Ubuntu 설치
CPU : 2개 설정
계정 : sist / 1234
```

### sudo 권한 등록
```bash
Ctrl + Alt + F3     # CLI 터미널 전환
# 로그인 : sist / 1234

sudo su -           # root 전환 (# 프롬프트로 변경)
# 비밀번호 : 1234

nano /etc/sudoers
```

```text
# sudoers 파일에 아래 줄 추가
root    ALL=(ALL:ALL) ALL
sist    ALL=(ALL:ALL) ALL   ← 이 줄 추가
```

```bash
Ctrl + O → Enter    # 저장
Ctrl + X            # 나가기
exit                # root 종료
```

### 로케일 설정 (한글 깨짐 방지)
```bash
sudo nano /etc/default/locale
```

```text
# 아래 내용 작성 (.으로 시작하는 부분 주의)
LANG=en_US.UTF-8
```

```bash
Ctrl + O → Enter → Ctrl + X

sudo locale-gen --purge   # 로케일 재생성
sudo reboot               # 재부팅
```

---

## 2. Java 21 설치 + 환경변수 설정

```bash
sudo apt-get update
sudo apt-get install openjdk-21-jdk

# 설치 경로 확인
# /usr/lib/jvm/java-21-openjdk-amd64

# 환경변수 등록
nano ~/.bashrc
```

```bash
# .bashrc 맨 아래에 추가
export JAVA_HOME=/usr/lib/jvm/java-21-openjdk-amd64
export PATH=$PATH:$JAVA_HOME/bin
```

```bash
Ctrl + O → Enter → Ctrl + X

source ~/.bashrc    # 환경변수 즉시 적용
java -version       # 설치 확인
```

---

## 3. Tomcat 11 설치 + 포트 설정

```bash
# tomcat.apache.org → Tomcat 11 → Core → tar.gz 링크 복사

sudo wget "https://downloads.apache.org/tomcat/tomcat-11/..../apache-tomcat-11.x.x.tar.gz"

ls -al              # 다운로드 확인

sudo tar -xvf apache-tomcat-11.0.x.tar.gz   # 압축 해제

ls -al              # 파란색 폴더 확인

cd apache-tomcat-11.0.x    # 탭 자동완성 활용 ⭐

sudo chmod 755 bin         # bin 폴더 실행 권한
sudo chmod 755 conf        # conf 폴더 읽기 권한

ls -al              # 권한 확인 (drwxr-xr-x 로 변경됨)
```

### 포트 80으로 변경
```bash
cd conf
sudo nano server.xml
```

```xml
<!-- 8080 → 80으로 변경 -->
<Connector port="80" protocol="HTTP/1.1" ...
```

```bash
Ctrl + O → Enter → Ctrl + X
cd ..
```

### Tomcat 실행/중지
```bash
sudo ./bin/startup.sh     # 구동 → localhost 브라우저 확인
sudo ./bin/shutdown.sh    # 중지
```

---

## 4. GitHub에서 프로젝트 clone + 배포

### Git 설치 + 계정 설정
```bash
sudo apt-get install git

git config --global user.name "atg8915"
git config --global user.email "ttajja69@gmail.com"
```

### 프로젝트 clone
```bash
git clone 리포지토리주소
cd
ls -al              # 클론된 폴더 확인
```

### classes 폴더 배포 (빌드 결과물 → WEB-INF로 이동)
```bash
cd apache-tomcat폴더
cd conf
sudo nano server.xml   # <Context> 태그 4종류 추가

cd ..
cd JspMVC프로젝트폴더
cd build

# classes 폴더를 WEB-INF로 복사
sudo cp -r classes ~/JspMVC프로젝트폴더/src/main/webapp/WEB-INF

# 확인
cd ../src/main/webapp/WEB-INF
ls -al              # classes 폴더 있으면 성공

# server.xml Context 경로 수정
cd
cd apache-tomcat폴더/conf
sudo nano server.xml   # Context docBase 경로 수정
```

---

## 5. Windows → GitHub 연동 (Git Bash)

### 최초 push (새 프로젝트)
```bash
cd C:/webDev/webStudy/JSPMVCTotalProject

git init                                              # git 저장소 초기화
git add .                                             # 전체 파일 스테이징
git branch -M main                                    # 브랜치명 main으로 변경
git remote add origin https://github.com/atg8915/JSPMVCTotalProject.git  # 원격 등록
git commit -m "first"                                 # 커밋
git push -u origin main --force                       # 강제 push (최초 1회)
```

### 이후 수정사항 push (일반)
```bash
git add .
git commit -m "변경내용 설명"
git push -u origin main
```

### 명령어 순서 요약
```text
최초 : init → add → branch -M main → remote add → commit → push --force
이후 : add → commit → push
```

---

## 6. nano 편집기 단축키 정리

| 단축키 | 기능 |
|--------|------|
| `Ctrl + O` → `Enter` | 저장 |
| `Ctrl + X` | 나가기 |
| `Ctrl + K` | 현재 줄 잘라내기 |
| `Ctrl + U` | 붙여넣기 |
| `Ctrl + W` | 검색 |

---

## 7. 전체 흐름 요약

```text
① Ubuntu VM 설치
   → sudo 권한 등록 → 로케일 설정 → 재부팅

② Java 21 설치
   → .bashrc 환경변수 등록 → source 적용

③ Tomcat 11 설치
   → tar.gz 다운로드 → 압축해제 → chmod 권한 → port 80 변경 → 실행

④ 프로젝트 배포
   → GitHub clone → classes 복사 → server.xml Context 설정

⑤ Windows에서 GitHub push
   → git init → add → commit → push
```
