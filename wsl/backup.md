# backup

## bashrc
``` shell
export PS1="\[\e[32;1m\][\t \u@\w]$>\[\e[0m\]"
export PATH=$PATH:/usr/local/go/bin
[ -f ~/proxy.sh ] && sh ~/proxy.sh
```
## vim
```
set nu
set tabstop=4
set wildmenu
```


## proxy
```
#!/bin/bash
host_ip=$(cat /etc/resolv.conf |grep "nameserver" |cut -f 2 -d " ")
export http_proxy="http://$host_ip:7890"
```