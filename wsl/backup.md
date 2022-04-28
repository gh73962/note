# backup

## bashrc
``` shell
export PS1="\[\e[32;1m\][\t \u@\w]$>\[\e[0m\]"
export PATH=$PATH:/usr/local/go/bin
export hostip=$(cat /etc/resolv.conf |grep -oP '(?<=nameserver\ ).*')
export https_proxy="http://${hostip}:7890"
export http_proxy="http://${hostip}:7890"
```
## vim
```
set nu
set tabstop=4
set wildmenu
```

## Other
nohup xxxx > ./nohup.out 2>&1 &