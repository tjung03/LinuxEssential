# Linux 기본 관리 실습 가이드

이 문서는 CentOS Stream 9 실습노트에서 다룬 작업 중 파일·권한·아카이브, Bash·프로세스, SSH를 다시 확인하기 위한 절차입니다. 코드 블록에는 셸 프롬프트를 넣지 않았으며, `$HOME`과 `$!`는 셸 변수입니다. 실제 경로와 계정은 자신의 VM에 맞춰 확인하세요.

실습 전에 `cat /etc/os-release`, `uname -r`로 OS와 커널을 확인하고, 필요한 명령의 옵션은 `man` 또는 `--help`에서 확인합니다. 학습 당시 환경과 다른 VM에서는 같은 명령이라도 출력과 기본 설정이 다를 수 있습니다.

## 파일과 권한

### 개인 실습 디렉터리에서 파일 확인

작업 위치를 확인하고 본인 홈 디렉터리 아래에 새 예제 디렉터리를 만듭니다. `lab_dir` 변수는 현재 셸에서 유지되므로 아래 명령을 같은 셸에서 순서대로 실행합니다. 새 디렉터리를 만들기 때문에 기존 실습 파일을 덮어쓰지 않습니다. 디렉터리 생성에 실패했다면 다음 명령을 실행하지 마세요.

```bash
pwd
lab_dir=$(mktemp -d "$HOME/linux-essential-lab.XXXXXX")
printf '실습 디렉터리: %s\n' "$lab_dir"
mkdir "$lab_dir/input" "$lab_dir/output"
printf 'INFO ready\nERROR sample\n' > "$lab_dir/input/sample.log"
ls -l "$lab_dir/input/sample.log"
```

`ls -l`에서 파일 종류·소유자·그룹·권한을 확인합니다. 실습노트의 `ls -l` 예시는 이 항목들을 읽고 `chmod`, `chown`, `chgrp`가 각각 무엇을 바꾸는지 구별하는 데 사용됩니다.

```bash
ln -s "$lab_dir/input/sample.log" \
  "$lab_dir/output/sample-link.log"
ls -l "$lab_dir/output/sample-link.log"
```

링크 파일과 원본 파일의 경로가 어떻게 표시되는지 확인합니다. 여기서는 대상 경로를 절대 경로로 만들었으므로 실습 디렉터리를 옮기면 링크가 유효하지 않을 수 있습니다.

```bash
chmod u=rw,g=r,o= "$lab_dir/input/sample.log"
ls -l "$lab_dir/input/sample.log"
```

예시의 소유자에게 읽기·쓰기, 그룹에 읽기 권한을 주고 기타 사용자의 권한은 제거합니다. 실제로 다른 사용자가 읽을 수 있는지는 파일 권한뿐 아니라 상위 디렉터리 접근 권한에도 좌우됩니다. `chown`으로 소유자를 바꾸는 작업은 해당 사용자·그룹과 필요한 권한을 먼저 확인한 다음 별도 실습 VM에서 진행하세요.

### 검색한 뒤 아카이브 생성·확인

파일 내용 검색과 파일 경로 검색은 다른 작업입니다.

```bash
grep -nEi 'warn|error|fail' "$lab_dir/input/sample.log"
find "$lab_dir" -type f -name '*.log' -print
tar -czf "$lab_dir/output/logs.tar.gz" \
  -C "$lab_dir/input" sample.log
tar -tzf "$lab_dir/output/logs.tar.gz"
```

첫 줄은 매칭된 **파일 내용과 줄 번호**, 둘째 줄은 조건에 맞는 **파일 경로**를 보여줍니다. `tar -tzf`로 아카이브 안의 경로를 확인한 다음, 필요할 때 별도의 디렉터리에 풉니다.

```bash
mkdir "$lab_dir/output/extracted"
tar -xzf "$lab_dir/output/logs.tar.gz" \
  -C "$lab_dir/output/extracted"
ls -l "$lab_dir/output/extracted"
```

외부에서 받은 아카이브라면 목록과 추출 경로를 먼저 확인하세요. 위의 절차는 직접 만든 `sample.log`를 대상으로 합니다.

텍스트 파일을 편집해야 한다면 `vim`에서 `i`로 입력 모드에 들어가고, `Esc` 뒤 `:wq`로 저장·종료합니다. 변경을 버리려면 `Esc` 뒤 `:q!`를 사용합니다. 파일 내용 차이는 `diff 이전파일 새파일`, 정렬 결과는 `sort 파일`로 확인할 수 있습니다. `vim`과 비교 명령도 노트에서 다루지만 별도의 자동화 파일은 필요하지 않습니다.

## Bash 환경과 프로세스

### 설정이 적용되는 셸 구분

실습노트에는 리다이렉션·파이프·변수·별칭과 `/etc/profile`, `~/.bash_profile`, `~/.bashrc` 등의 시작 파일이 나옵니다. 로그인 셸인지, 대화형 비로그인 셸인지에 따라 읽는 파일이 달라집니다. 배포판의 시작 파일은 다른 파일을 추가로 불러올 수도 있으므로, 노트에 적힌 순서를 모든 환경의 고정 규칙으로 적용하지 않습니다.

```bash
printf '%s\n' "$PATH"
type grep
alias
grep -Ei 'warn|error|fail' "$lab_dir/input/sample.log" | sort
```

앞 절의 `printf ... > 파일`은 출력을 파일로 보내는 리다이렉션이고, 마지막 줄의 `|`는 `grep`의 출력을 `sort`의 입력으로 전달하는 파이프입니다. `env/bashrc.txt`는 당시 사용자 설정을 담은 **참고 파일**입니다. `/test`를 `PATH`에 추가하고 파일 삭제에 영향을 주는 별칭도 있으므로 전체 내용을 현재 계정에 복사하거나 바로 `source`하지 마세요. 필요한 설정 한 줄의 의미를 확인한 뒤 본인 시작 파일을 백업하고 적용해야 합니다.

### 시작한 프로세스의 상태 확인

다음 예시는 사용자 권한으로 짧게 실행되는 `sleep` 프로세스를 대상으로 합니다. 동일한 셸에서 순서대로 실행합니다.

```bash
sleep 120 &
job_pid=$!
jobs
ps -p "$job_pid" -o pid,ppid,stat,cmd
kill -TERM "$job_pid"
ps -p "$job_pid" -o pid,stat,cmd
```

`jobs`는 현재 셸의 작업, `ps`는 지정한 PID의 프로세스 상태를 봅니다. `TERM`을 보낸 직후에는 종료 처리가 아직 진행 중일 수 있습니다. 시스템 전체의 사용량과 프로세스를 살피는 실습에는 `top`을 사용할 수 있으며, 종료는 `q`입니다. 기존 `bin/cpu*.sh`는 CPU 부하를 만드는 코드이므로 이 기본 확인 절차에 필요하지 않습니다.

## SSH 접속과 파일 전송

이 절은 접속 권한이 있는 별도 실습 VM이 준비된 경우에만 사용합니다. `사용자명`과 `서버주소`를 실제 값으로 바꾸고 대상이 맞는지 확인합니다. 최초 접속에서 호스트 키 지문을 묻는다면 관리자가 알려준 값 또는 신뢰할 수 있는 경로의 값과 대조한 뒤 수락합니다. 앞 절에서 만든 `lab_dir` 변수도 같은 셸에서 사용합니다.

```bash
ssh 사용자명@서버주소 'hostname'
remote_dir=$(ssh 사용자명@서버주소 'mktemp -d "$HOME/linux-essential-lab.XXXXXX"') &&
scp "$lab_dir/input/sample.log" "사용자명@서버주소:$remote_dir/sample.log" &&
ssh 사용자명@서버주소 "ls -l -- '$remote_dir/sample.log'"
```

첫 명령은 원격에서 호스트 이름을 확인합니다. 다음에는 원격 홈 아래에 별도 디렉터리를 만들어 파일을 전송하고 결과를 확인합니다. `remote_dir`는 현재 셸에서 유지되며, `&&`는 앞 명령이 실패하면 다음 명령을 실행하지 않게 합니다. 실습노트와 `bin/cmd.sh`의 주소는 당시 VM 네트워크의 값입니다. `bin/cmd.sh`는 한 명령을 세 호스트에 전달하므로 대상과 명령의 영향을 확인하기 전에는 실행하지 마세요.

## 확인 범위

위 예시는 학습 주제를 다시 따라가기 위한 재구성 절차입니다. 저장소에 측정 결과나 각 VM에서 실행한 로그가 포함되어 있지 않으므로, 특정 환경에서의 성공 결과나 성능 수치를 전제로 하지 않습니다. 스크립트의 Bash 문법을 점검하려면 저장소 루트에서 `bash -n bin/*.sh`를 사용할 수 있지만, 이는 명령의 실제 동작까지 검증하지 않습니다.
