# LinuxEssential

CentOS Stream 9 가상 머신에서 다룬 Linux 기본 관리 실습을 정리한 저장소입니다. 파일과 권한을 확인하고, Bash 환경과 프로세스를 살펴본 뒤, SSH로 다른 VM에 접속해 파일을 전송하는 흐름을 다룹니다. Bash 설정 예시와 보조 스크립트도 함께 있습니다.

## 다루는 범위

| 영역 | 핵심 내용 | 다시 확인할 것 |
| --- | --- | --- |
| 파일과 디렉터리 | 경로, 파일 종류, 링크, 소유권·권한·ACL | `pwd`, `ls -l`, `find`, `chmod`, `getfacl`로 대상과 변경 결과 확인 |
| 검색과 보관 | `grep`으로 내용 검색, `tar`로 묶고 압축 | 아카이브 목록을 먼저 확인한 뒤 별도 디렉터리에 풀기 |
| Bash 환경 | 리다이렉션, 파이프, 변수·별칭, 시작 파일 | 현재 셸과 새 셸에서 설정이 적용되는 범위 구분 |
| 프로세스 | 전경·배경 작업, 상태 조회, 신호, 자원 관찰 | `jobs`, `ps`, `top`과 종료 후 상태 확인 |
| 원격 작업 | SSH 접속, 원격 명령, `scp` 파일 전송 | 대상 호스트 식별 후 전송된 파일 확인 |

시스템 정보를 살피는 `man`, `uname`, `/etc/os-release`, 텍스트 편집기 `vim`, 파일 비교 명령도 기본 실습에 포함됩니다. [기본 실습 가이드](docs/practice-guide.md)는 그중 다시 따라가기 좋은 명령의 적용 위치와 확인 방법을 연결합니다.

## 실습 환경과 적용 범위

학습노트에는 CentOS Stream 9 기반 `main`, `server1`, `server2` VM과 `192.168.10.0/24` 실습 네트워크가 기록되어 있습니다. 아래 스크립트 중 일부는 그 당시 IP 주소와 `/root/bin` 경로를 그대로 사용합니다. **다른 시스템에서 바로 실행할 수 있는 범용 운영 도구로 취급하지 마세요.**

문서의 로컬 파일 예시는 본인 홈 디렉터리 아래에 새 실습 디렉터리를 만든 뒤 확인하도록 작성했습니다. SSH 예시는 접속 가능한 별도 VM과 유효한 계정이 있을 때만 적용됩니다. VM 이름, 인터페이스 이름, 패키지 상태, 접근 권한은 각 환경에서 확인해야 합니다.

## 저장소 구성

```text
.
├── README.md
├── docs/
│   └── practice-guide.md     # 파일·셸·프로세스·SSH 실습과 확인 절차
├── env/
│   └── bashrc.txt            # Bash 설정 예시
└── bin/                      # 11개의 보조 스크립트
```

### 기존 파일을 읽을 때

| 파일 | 코드에서 확인되는 동작 | 실행 전 확인할 사항 |
| --- | --- | --- |
| [`bin/chklog.sh`](bin/chklog.sh) | 전달받은 로그 파일에서 **현재 날짜**의 경고·오류 관련 단어를 검색 | 로그 파일 접근 권한과 날짜·언어 형식 확인. 표시되는 줄 번호는 날짜 필터 결과 기준이며 원본 파일의 줄 번호가 아님 |
| [`bin/cmd.sh`](bin/cmd.sh) | 정해진 세 IP 주소에 원격 명령을 순서대로 전달 | 세 주소와 명령의 영향 확인. 인수를 따옴표 없이 `$*`로 전달하므로 공백·인용이 필요한 명령에 주의 |
| [`bin/cpu.sh`](bin/cpu.sh), [`cpu2.sh`](bin/cpu2.sh), [`cpu3.sh`](bin/cpu3.sh) | 반복 계산으로 CPU 부하를 만들거나 `cpu.sh`를 백그라운드에서 시작 | 테스트 VM의 자원 상태 확인. `cpu3.sh`는 현재 디렉터리의 `./cpu.sh`에 의존하며, 정상 종료 뒤에도 시작한 작업이 남을 수 있고 인터럽트 시 `killall cpu.sh`를 실행 |
| [`bin/disk.sh`](bin/disk.sh) | `dnf`로 패키지를 설치하고 `fio`로 `/tmp/testfile`에 임의 쓰기 후 해당 파일 삭제 | 관리자 권한, `/tmp` 여유 공간, 저장 장치 영향, 같은 경로의 기존 파일 여부 확인 |
| [`bin/net_load.sh`](bin/net_load.sh) | `iperf3` 서버 또는 클라이언트를 실행하고 클라이언트 측에서 결과 파일·그래프 생성 시도 | 서버 IP와 의존 명령 확인. 모드 검사 전에 `/root/bin/logs`와 `/root/bin/plots`를 삭제하고 다시 만듦 |
| [`bin/env_conf.sh`](bin/env_conf.sh), [`env_unconf.sh`](bin/env_unconf.sh) | 시스템·사용자 Bash 시작 파일을 편집하고 `/etc/profile.d/test.sh`를 생성·삭제 | 기존 설정 백업과 변경 범위 확인. 복원 스크립트는 표시 문자열이 포함된 다른 줄도 삭제할 수 있음 |
| [`bin/poweroff.sh`](bin/poweroff.sh) | 지정된 세 IP 주소로 SSH 접속해 `poweroff` 실행 | 대상 시스템을 종료해도 되는지 먼저 확인 |
| [`bin/testscript.sh`](bin/testscript.sh) | `cowsay`, `boxes`를 이용한 출력 예시 | 두 명령의 설치 여부 확인 |
| [`env/bashrc.txt`](env/bashrc.txt) | `PATH`, 프롬프트, 여러 별칭을 지정한 사용자 설정 예시 | 전체 파일을 `~/.bashrc`에 덮어쓰지 말 것. `/test` 경로와 삭제 명령 별칭도 포함 |

스크립트 파일의 존재는 해당 파일을 현재 환경에서 실행·검증했다는 의미가 아닙니다. 특히 부하 생성, 설정 파일 편집, 파일 삭제, 원격 종료 스크립트는 실행 예제 대신 **동작을 읽고 영향 범위를 판단할 자료**로 다룹니다. `bash -n bin/*.sh`는 Bash 문법을 확인하는 명령일 뿐, 실제 동작의 안전성이나 성공을 증명하지 않습니다.

## 기본 실습 흐름

1. [파일·권한·검색·아카이브](docs/practice-guide.md#파일과-권한): 개인 실습 디렉터리에서 파일을 만들고 `umask`, 기본 권한, ACL을 구분해 확인합니다.
2. [Bash 환경과 프로세스](docs/practice-guide.md#bash-환경과-프로세스): 현재 셸의 설정과 작업 상태를 구분합니다.
3. [SSH와 파일 전송](docs/practice-guide.md#ssh-접속과-파일-전송): 대상 호스트를 식별한 뒤 파일을 전송하고 확인합니다.

### 참고 자료

- [Red Hat Enterprise Linux 9: Configuring basic system settings](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/htmlsingle/configuring_basic_system_settings/index): 권한과 시스템 설정을 확인할 때 참고할 수 있는 공식 문서
- [GNU Bash: Startup Files](https://www.gnu.org/software/bash/manual/html_node/Bash-Startup-Files.html): 셸 시작 방식에 따른 설정 파일의 읽기 순서
