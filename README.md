# Xtra-Schedule-Backup
Xtrabackup Schedule Backup

<code>
<pre>
#!/bin/sh

# Xtra Configure
xtra_home=/mysql/xtrabackup/bin/innobackupex

# DB Configure
back_user=tr_back
back_pass=
back_dir=/backup
backlog_dir=/backup
db_config=/mysql/mysql/my.cnf
db_socket=/tmp/mysql.sock
fix_name='QA'

# SSH SETTING(Need ssh rsa key add)
ssh_user=mysql
storage_ip=
storage_dir=/backup

# FIX Setting
DAYS_NO=$(LANG=C date +%w)
#DAYS_NO=1
BASENO=`expr $DAYS_NO - 1`

# Backup Scheduler
case $DAYS_NO in
    0)
        # Full Backup Code
        $xtra_home --default-file=$db_config --host=localhost --socket=$db_socket \
        --slave-info \
        --parallel=4 \
        --user=$back_user --password=$back_pass \
        --no-timestamp \
        $back_dir/Export_${fix_name}_0${DAYS_NO} > $backlog_dir/Export_${fix_name}_0${DAYS_NO}.log 2>& 1
    ;;
    1|2|3|4|5|6 )
        # TO LSN READ
        to_lsn=$(cat $back_dir/Export_${fix_name}_0${BASENO}_lsn)
        # Incremental Backup Code
        $xtra_home --default-file=$db_config --host=localhost --socket=$db_socket \
        --slave-info \
        --parallel=4 \
        --user=$back_user --password=$back_pass \
        --incremental --incremental-lsn=${to_lsn} \
        --no-timestamp \
        $back_dir/Export_${fix_name}_0${DAYS_NO} > $backlog_dir/Export_${fix_name}_0${DAYS_NO}.log 2>& 1
    ;;
esac

# Make LSN FILE
cat $back_dir/Export_${fix_name}_0${DAYS_NO}/xtrabackup_checkpoints |grep to_lsn|awk '{print $3}' > $back_dir/Export_${fix_name}_0${DAYS_NO}_lsn

# Compress & Transfer Backup
tar -cvzf $back_dir/Export_${fix_name}_0${DAYS_NO}.tar.gz $back_dir/Export_${fix_name}_0${DAYS_NO} >> $backlog_dir/Export_${fix_name}_0${DAYS_NO}.log 2>& 1
scp $back_dir/Export_${fix_name}_0${DAYS_NO}.tar.gz ${ssh_user}@${storage_ip}:${storage_dir} >> $backlog_dir/Export_${fix_name}_0${DAYS_NO}.log 2>& 1
rm -rf $back_dir/Export_${fix_name}_0${DAYS_NO} $back_dir/Export_${fix_name}_0${DAYS_NO}.tar.gz
</pre>
</code>
