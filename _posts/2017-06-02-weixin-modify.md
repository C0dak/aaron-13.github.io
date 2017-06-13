# 微信升级后的相关操作
------
周五收到微信公众号升级的通知，没有在意，觉得跟我没啥关系。直到周日的时候，发觉微信怎么一直不报警了，不应该啊，按理说，没有报警不是更好。这和我维护的服务器状况不符，虽说不是老司机，但至少它们的日常运行，系统状态，资源等情况还是了解的，有些特定的服务器(db数据库等)会在某个时间段运行一些crontab，会导致网络，磁盘I/O，cpu性能等指数的飙升，触发报警，但是周末没动静了，再查看邮箱，有告警邮件，这下，微信的升级还真和我有关系了。公司的微信公众号有专门的运营，我索性就自己申请了一个用来告警。只能去看更新变动了。还好，只是板块位置变换和一些小改动。

查看了一下，之前的secretID是不能再用了，更新到一个新的secretID。替换成新的secretID，然后执行脚本，报错了`{"errcode":40014,"errmsg":"invalid access_token"}`,什么鬼，token无效，重新生成一个新的secretID，还是报错，那就不是secretID的问题喽。索性把脚本里的命令手动执行一遍

**获取token**
`curl -s -G "https://qyapi.weixin.qq.com/cgi-bin/gettoken?corpid=XXXXXXXXXXXXXXXXXXXXX&corpsecret=XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"`

得到json格式的结果
`{"errcode":0,"errmsg":"ok","access_token":"XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX","expires_in":7200}`

这不是OK的么，继续排查，是不是在传递token的时候出错了，检查脚本，还真发现了:
`Gtoken=$(/usr/bin/curl -s -G $GURL | awk -F\" '{print $4}')`
$4="errmsg"并不是想要的token，修改为
`Gtoken=$(/usr/bin/curl -s -G $GURL | awk -F\" '{print $10}')`
重新执行脚本，OK。
还好其他不要怎么变动。虽然其中还走了一些弯路，都是自己挖的坑。

------

```Shell
#!/bin/bash

GURL="https://qyapi.weixin.qq.com/cgi-bin/gettoken?corpid=XXXXXXXXXXXXXXXXXX&corpsecret=XXXXXXXXXXXXXXXXXXXXXXXXXXX"
Gtoken=$(/usr/bin/curl -s -G $GURL | awk -F\" '{print $10}')

PURL="https://qyapi.weixin.qq.com/cgi-bin/message/send?access_token=$Gtoken"

function body() {
        local int AppID=1
        local UserID=$1
        local PartyID=1
        local Msg=$(echo "$@" | cut -d" " -f3-)
        printf '{\n'
        printf '\t"touser": "'"$UserID"\"",\n"
        printf '\t"toparty": "'"$PartyID"\"",\n"
        printf '\t"msgtype": "text",\n'
        printf '\t"agentid": "'" $AppID "\"",\n"
        printf '\t"text": {\n'
        printf '\t\t"content": "'"$Msg"\""\n"
        printf '\t},\n'
        printf '\t"safe":"0"\n'
        printf '}\n'
}
/usr/bin/curl --data-ascii "$(body $1 $2 $3)" $PURL
```
