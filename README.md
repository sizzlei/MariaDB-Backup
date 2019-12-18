# MariaDB-Backup
xtrabackup 툴을 사용하거나, MariaBackup 툴을 사용하는 경우 full backup 과 incremental 백업을 할수 있는 스크립트

## Configure 
+ dbUser
  - 백업 사용 계정(Database 접근 계정)
+ dbPass
  - 백업 계정 패스워드
+ dbSocket
  - Database 연결 Socket
+ privUser
  - 백업 완료 후 해당 사용자로 백업본의 소유자를 지정
+ mysqlPath
  - Database 의 BaseDir
+ xtraPath
  - backupXDB의 경우 xtrabackup을 사용하므로 xtrabakup 경로 지정
+ configPath
  - mariadb의 Config 위치를 지정
+ backupPath
  - 백업이 저장되는 경로
+ backupName
  - 백업시 생성되는 디렉터리 Prefix
+ logPath
  - 백업 로그 경로
+ parallelCore
  - 백업시 사용할 thread의 수
+ runDiv
  - [prod/dev]옵션이며 prod는 운영환경에서 지정하는 경우 이며 Crontab 또는 기타 솔루션에서 주기적 실행을 목적으로 작성, Incremental Backup이 기본적으로 지정되어있으며, 요일별로 진행된다. 
  - 일요일 : Full
  - 그외 incremental

+ dev를 사용하는 경우 원하는 백업을 진행하거나 full backup만 지정 할 수 있다. 


## backupXDB (Xtrabackup)
<code><pre>
runDiv [Dev] full Backup :
./backupXDB full

runDiv [Dev] Incremental Backup :
./backupXDB inc baseDirNumber
ex) Full Backup Base Incremental
./backupXDB inc 0

Restore Usage:
Important.
Data Directory And binary Log Directory Cleaning.
Full Backup Restore:
1. innobackupex --defaults-file=$Configdir --apply-log $fullbackupdir
2. innobackupex --defaults-file=$Configdir --copy-back $fullbackupdir

Incremental Backup Restore:
1. innobackupex --defaults-file=$Configdir --apply-log --redo-only $fullbackupdir
2. innobackupex --defaults-file=$Configdir --apply-log --redo-only $fullbackupdir --incremental-dir=$incbakcupdir
3. innobackupex --defaults-file=$Configdir --copy-back $fullbackupdir
</pre></code>

## backupMDB (Mariabackup)
<code><pre>
runDiv [Dev] full Backup :
./backupMDB full

runDiv [Dev] Incremental Backup :
./backupMDB inc baseDirNumber
ex) Full Backup Base Incremental
./backupMDB inc 0

Restore Usage:
Important.
Data Directory And binary Log Directory Cleaning.
Full Backup Restore:
1. mariabackup --defaults-file=$Configdir --prepare --target-dir=$fullbackupdir
2. mariabackup --defaults-file=$Configdir --copy-back --target-dir=$fullbackupdir

Incremental Backup Restore:
1. mariabackup --defaults-file=$Configdir --prepare --target-dir=$fullbackupdir
2. mariabackup --defaults-file=$Configdir --prepare --target-dir=$fullbackupdir --incremental-dir=$incbakcupdir
3. mariabackup --defaults-file=$Configdir --copy-back --target-dir=$fullbackupdir

But Mariadb Version 10.2 >
Full Backup Restore
1. mariabackup --defaults-file=$Configdir --prepare --target-dir=$fullbackupdir
2. mariabackup --defaults-file=$Configdir --copy-back --target-dir=$fullbackupdir

Incremental Backup Restore:
1. mariabackup --defaults-file=$Configdir --prepare --apply-log-only --target-dir=$fullbackupdir
2. mariabackup --defaults-file=$Configdir --prepare --apply-log-only --target-dir=$fullbackupdir --incremental-dir=$incbakcupdir
3. mariabackup --defaults-file=$Configdir --copy-back --target-dir=$fullbackupdir
</pre></code>
