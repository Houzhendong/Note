1. 查看端口占用

   linux：`lsof -i:<port>` / `netstat -tunlp | grep <port>`

   windows：`netstat -aon|findstr "<port>"`