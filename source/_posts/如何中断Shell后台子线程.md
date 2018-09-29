---
title: 如何中断Shell后台子线程
comments: false
date: 2018-09-29 16:34:41
categories: linux
tags: shell
img:
---
# 如何中断Shell后台子线程
<!-- TOC -->

- [如何中断Shell后台子线程](#如何中断shell后台子线程)
    - [背景](#背景)
    - [条件](#条件)
    - [shelll脚本步骤](#shelll脚本步骤)
        - [查找匹配的PID](#查找匹配的pid)
        - [验证应用状态](#验证应用状态)
        - [输出部署日志](#输出部署日志)
        - [入口](#入口)
        - [执行Shell](#执行shell)
    - [改造思考](#改造思考)

<!-- /TOC -->

## 背景
当我们部署一个应用服务的时候，通常需要登录控制台看日志，实时的看可以用命令打印 `-f`。  
但是对于自动化来说，希望可以通过部署的控制台看日志信息，像`Jeckins BlueOcean`或者是`GitLab CI/CD`都可以直接查看Job的动态情况。
对于后台日志进程打印不需要的时候需要关闭，接下来通过自定义shell实现。

## 条件
本文使用`Kubernetes`环境下日志输出作为案例。
部署镜像`chaoscoffee/springcloud-jpa`(可以拉取`docker pull chaoscoffee/springcloud-jpa`)
部署容器名称`springcloud-jpa-deployment`.

## shelll脚本步骤

### 查找匹配的PID
这里分步来说明下:
1. 先过滤出使用的日志命令`$1`后台进程。
2. 反向查找需要匹配的线程`grep -v grep`,也就是过滤掉包含有`grep`字符的行。
3. 格式化输出`awk`需要的进程信息,即`PID`。
4. 以上步骤找到了我们的后台存在的日志进程，下面就是`kill`完成。
5. `$!`是Shell最后运行的后台Process的PID。使用`trap`信号指令干掉它，脚本退出时候触发。（这一步不是必需，这是为了安全退出日志脚本做了一层校验）


``` sh
function killLogs() {
    #是否存在进程
    LAST_PID=$(ps -ef|grep "$1"|grep -v grep|awk '{print $'$2'}')
    echo "LAST_PID=$LAST_PID"
    if [ -n "$LAST_PID" ] &&  [ "$LAST_PID" -gt 0 ]; then  
        echo "LAST PROCESS NOT EXIT, NOW KILL IT!"  
        kill $LAST_PID  
        sleep 1  
    fi 
    #杀死当前进程
    THIS_PID=$!
    if  [ ! -n $THIS_PID ] ;then
        echo "THIS_PID=$THIS_PID"
        trap "kill $THIS_PID" TERM  
        wait
    fi
}
```


### 验证应用状态
这个步骤`验证`，比较简单，主要是与上一个步骤配合使用，为了查询部署Pod的状态是否成功，以便作为结束日志打印的条件。
1. 查询`Pod`状态并且输出。
2. 返回定义的状态，注意shell函数的返回值只能是0-255。

``` sh
function validationStatus() {
    STATUS=$(kubectl get pods -l app=$1|awk 'NR!=1{print $3}')
    arr=($STATUS)
    result=0
    for s in ${arr[@]}
    do
        echo $s
        if [ "$s" = "Pending" -o "$s" = "ContainerCreating" -o "$s" = "Terminal" ]
        then 
            return 0
        elif [ "$s" = "Running" ]
        then
            result=1
        else
            return 2
        fi        
    done
    return $result
}
```

### 输出部署日志
这个步骤是为了结束打印后，控制台可能日志显示不全的控制，打印日志是实时的，中间查询部署状态会阻塞住进程，会导致打印不完全。  
**例如**:  
日志输出到某一行之后，中间查询状态是成功的，这时候shell退出了，但是日志并未完全输出。

``` sh
function finishJob() {
    echo $1
    echo $2
    killLogs "kubectl logs" 1
    echo "查询1h前日志: "
    sleep 3
    kubectl logs $3 --since=1h
}
```

### 入口
1. 实现输出日志并且读取每行。
2. 验证应用状态，对应于上面第二步骤。
3. 流程控制，获取到应用部署状态成功或者失败退出循环，否则继续打印日志。

``` sh
echo "部署日志记录中..."
kubectl logs $1 -f|while read line
do
  validationStatus "$2"
  OK=$?
  if [ $OK == 1 ]
  then
      finishJob "$line" "-> Job执行成功!!!" "$1"
      break
  elif [ $OK == 2 ]
  then	  
      finishJob "$line" "-> Job执行失败!!!" "$1"
      exit 1
  else
      echo $line
  fi
done
```

### 执行Shell
定义一个shell,叫做`log.sh`.
1. 赋予执行权限
2. 执行  
3. 打印状态

``` sh
chmod +x logs.sh
./logs.sh "deployment/springcloud-jpa-deployment" "springcloud-jpa"
echo $?
```

## 改造思考
如果只想在日志打印中判断部署成功失败，那就可以采用以下脚本，这样就略去重新调用其他接口的步骤，通过日志内容关键字判断。
1. 匹配成功或者失败的标识。
2. 做出相应操作。

``` sh
echo "部署日志记录中..."
kubectl logs $1 -f|while read line
do
  OK=`echo $line|grep "8089"|wc -l` #匹配成功标识
  FAIL=`echo $line|grep "Exception"|wc -l` #匹配失败标识
  if [ $OK == 1 ]
  then
      finishJob "$line" "-> Job执行成功!!!" "$1"
      break
  elif [ $FAIL == 1 ]
  then	  
      finishJob "$line" "-> Job执行失败!!!" "$1"
      exit 1
  else
      echo $line
  fi
done
```