# Docker

## 定时清理containers log file
```
#!/bin/sh

path=/var/lib/docker/containers
# 10M=10*1024*1024=10485760
filesizelimit=10240

for logfile in "$path"/*/*.log
do
  echo "$logfile"
  actualsize=$(wc -c <"$logfile")
  if [ $actualsize -ge $filesizelimit ]; then
    echo size is over $actualsize bytes
    `echo "">$logfile`
  else
    echo size is under $actualsize bytes
  fi
done
```
linux定时任务-`crontab`

![](./crontab.png)

```
# Run the daily jobs
* 1 * * * root /etc/init.d/smb restart
```
