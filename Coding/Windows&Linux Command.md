1. 查看端口占用

   linux：`lsof -i:<port>` / `netstat -tunlp | grep <port>`

   windows：`netstat -aon|findstr "<port>"`

2. WSL2中添加代理  
   > `export all_proxy="socks5://{windows ip}:port"`  
   > `export ALL_PROXY="socks5://${windows ip}:port"`  
   取消代理
   > `unset all_proxy`
   > `unset ALL_PROXY`