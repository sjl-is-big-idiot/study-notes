



### 批量查看各节点运行的进程

```bash
#!/bin/bash

for host in `cat /home/hadoop/workers`
do
  echo "###################################### $host ######################################"
  ssh $host "jps"
done
```

