server-ctl
启动服务：
```shell
./server-ctl start
```
关闭服务:
```shell
./server-ctl shutdown
```

热更新模块
```shell
./server-ctl restart task_server
./server-ctl restart task_server luck_server uuid_server
```
cowboy router 更新，需要在 application 里面实现 reload 函数
```shell
./server-ctl graceful
```
进入当前运行的服务节点内
```shell
./server-ctrl debug
```


```shell
#!/usr/bin/env bash
APPNAME="web_agent_app" ##对应 application 文件
NODENAME="webagent"

ERL_A="+A 64"     ## 异步线程池大小， 默认为0
ERL_a="+a 64"     ## 系统栈大小， 默认16KB
ERL_Q="+Q 102400" ## PORT总数限制 +Q 1024
ERL_P="+P 102400" ## 最大并发进程数
ERL_S="" #+stbt db +sbwt none +sub true" ## 保证每个调度器业务平均
ERL_FORK="" #%-env ERL_NO_VFORK 1" ##改用fork open_port
ERL_PA="-pa _build/default/lib/*/ebin"

ERL_COOKIE="vmq"
ERL_CONFIG="priv/app.config"
ERL="erl -setcookie $ERL_COOKIE -config $ERL_CONFIG"
HOST="127.0.0.1"  #`hostname|sed 's/wx-//g'|sed 's/-/./g'`

FLAG="'"
NODE="$NODENAME@$HOST"

## Color
RED='\e[0;31m'
GREEN='\e[0;32m'
BLUE='\e[0;34m'
CYAN='\e[0;36m'
NC='\e[0m'


case $1 in

  start)

        StartArg="$ERL $ERL_PA $ERL_Q $ERL_P $ERL_A $ERL_a $ERL_S -s $APPNAME +K true -name $NODENAME@$HOST -detached"
        $ERL $ERL_PA -noshell -hidden -name manage@$HOST \
            -eval "server_manage:start('$NODENAME@$HOST',\"$StartArg\")"
     ;;
  startconsole)
        echo -e "Check node:[$BLUE$NODENAME@$HOST$NC]"
        RESULT=$($ERL $ERL_PA -noshell -hidden -name servermanage@$HOST -eval "server_manage:check({'$NODENAME@$HOST'})" -s init stop)
        if [ "$RESULT" = "false" ]; then
                echo -e "Start node:[$BLUE$NODENAME@$HOST$NC]"
                $ERL $ERL_PA $ERL_Q $ERL_P $ERL_A $ERL_a $ERL_S +K true -s $APPNAME -name $NODENAME@$HOST
        else
                echo -e "$RED[ERROR:Application $NODENAME@$HOST already startd]$NC"
        fi
      ;;
  graceful)
        $ERL $ERL_PA -noshell -hidden -name manage@$HOST \
                -eval "server_manage:restart({'$NODENAME@$HOST',$APPNAME,$APPNAME})"
        $ERL $ERL_PA -noshell -hidden -name manage@$HOST \
                -eval "server_manage:execute('$NODENAME@$HOST',$APPNAME,reload,[])"
       ;;
  restart)
        for((i=2; i<=$#; i++)); do
            MODULE=${!i}
            if [ ! -n "$MODULE" ]; then
                echo -e "$RED Usage: $0 restart MODULENAME $NC"
            else
                $ERL $ERL_PA -noshell -hidden -name manage@$HOST \
                -eval "server_manage:restart({'$NODENAME@$HOST',$APPNAME,$MODULE})"
            fi
        done
        ;;
  shutdown)
        $ERL $ERL_PA -noshell -hidden -name servermanage@$HOST \
             -eval "server_manage:shutdown({'$NODENAME@$HOST',$APPNAME})"
        ;;
  status)
          $ERL $ERL_PA -noshell -hidden -name servermanage@$HOST \
               -eval "server_manage:status({'$NODENAME@$HOST'})"
        ;;
  debug)
        echo "Press any key to continue"
        $ERL  $ERL_PA -name servermanage@$HOST  -remsh $NODENAME@$HOST \
        ;;
  *)
        echo ""
        echo -e "$CYAN Server Control $NC Usage: Hello World"
        echo -e "---------------------------------------------------------------------------------"
        echo -e " $0$GREEN start$NC             Start application in background"
        echo -e " $0$GREEN startconsole$NC      Start application in foreground, input $CYAN q() $NC to quit"
        echo -e " $0$GREEN shutdown$NC          Shutdown the application"
        echo -e "---------------------------------------------------------------------------------"
        echo -e " $0$GREEN graceful$NC          Restart $GREEN$APPNAME$NC and reload http router"
        echo -e " $0$GREEN restart MODULE$NC    Restart MODULE , dont't include .erl "
        echo -e " $0$GREEN restart MODULE1 MODULE2 MODULE3 ... $NC"
        echo -e " $0$GREEN status$NC            Check node's status"
        echo -e " $0$GREEN debug$NC             Connect to Node's shell , $RED Notice$NC: use $CYAN CTRL + g $NC , input $CYAN q $NC to quit"
        echo ""
        exit 1
esac

exit 1
```


server_manage.erl

```erlang
-module(server_manage).
-export([check/1, start/2, shutdown/1, restart/1, status/1, execute/4]).

check({Node}) ->
        case net_kernel:connect_node(Node) of
                true -> io:format("true");
                false -> io:format("false")
        end.

status({Node}) ->
        case net_kernel:connect_node(Node) of
                true ->
                        case rpc:call(Node, erlang, is_alive, []) of
                                true ->
                                        io:format("Node: [\e[32m~s\e[00m]  [\e[32m is Running \e[00m]  ... ", [Node]),
                                        io:format("\e[01;32mok\e[00m~n");
                                _ ->
                                        io:format("Node: [\e[32m~s\e[00m] [\e[31m has Shutdown \e[00m]   ...", [Node])
                        end;
                false ->
                        io:format("fail~n")
        end,
        halt().

shutdown({Node, Service}) ->
        case net_kernel:connect_node(Node) of
                true ->
                        io:format("Application: [\e[32m~s\e[00m] Stop...", [Service]),
                        case rpc:call(Node, application, stop, [Service]) of
                                {badrpc, nodedown} ->
                                        io:format("[\e[01;31mERROR: application not start...\e[00m]~n");
                                {badrpc, _} ->
                                        io:format("\e[01;31mfail\e[00m~n"),
                                        io:format("Node: [\e[32m~s\e[00m] Shutdown ... ", [Node]),
                                        rpc:call(Node, erlang, halt, [0]),
                                        io:format("\e[01;32mok\e[00m~n");
                                _ ->
                                        io:format("\e[01;32mok\e[00m~n"),
                                        io:format("Node: [\e[32m~s\e[00m] Shutdown ... ", [Node]),
                                        rpc:call(Node, init, stop, []),
                                        io:format("\e[01;32mok\e[00m~n")
                        end;
                false ->
                        io:format("[\e[01;31mERROR\e[00m] Can't connect [\e[01;36m~p\e[00m] ...~n", [Node])
        end,
        halt().

execute(N, M, F, A) ->
        case net_kernel:connect_node(N) of
                true ->
                        case rpc:call(N, M, F, A) of
                                {badrpc, nodedown} ->
                                        io:format("[\e[01;31mERROR: application not start...\e[00m]~n");
                                _ ->
                                        io:format("\e[0;32m Execute ~p:~p(~p) ok ...\e[00m~n", [M, F, A])
                        end;
                false ->
                        io:format("[\e[01;31mERROR\e[00m] Can't connect [\e[01;36m~p\e[00m] ...~n", [N])
        end,
        halt().

start(Node, StartArg) ->
        case net_kernel:connect_node(Node) of
                true -> io:format("[\e[01;31mERROR\e[00m] [\e[01;36m~p\e[00m] already start...~n", [Node]);
                false ->
                        io:format("[\e[32mExecute\e[00m]\e[36m~s\e[00m ... ", [StartArg]),
                        os:cmd(StartArg),
                        io:format("\e[32mok\e[00m~n")
        end,
        halt().

restart({Node, Service, Module}) ->
        case net_kernel:connect_node(Node) of
                true ->
                        _Ret = rpc:call(Node, code, purge, [Module]),
                        io:format("~p: [\e[31m~s\e[00m] Reload Module [\e[31m~s\e[00m]...", [Node, Service, Module]),
                        Ret2 = rpc:call(Node, code, load_file, [Module]),
                        io:format("\e[01;32m~p\e[00m~n", [Ret2]);
                false ->
                        io:format("fail~n")
        end,
        halt().
```
