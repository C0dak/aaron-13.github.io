# logstash切分log4j格式的日志

------



tomcat的日志被log4j配置文件定义成特定格式，使用logstash的filter插件将其分割成所需的格式。

log4j的PatternLayout参数含义

|参数|含义|示例|
|:--:|:--:|:--:|
| %c | 列出logger名字空间的名称| %c a.b.c |
| %C | 列出调用logger的类的全名| %C org.apache.xyz.SomeClass |
| %d | 显示日志记录时间，{<日期格式>}使用ISO8601定义的日期格式| %d{ISO8601} 2017-09-08 11:11:11,111 |
| %F | 显示调用logger的源文件名| %F MyClass.java |
| %l | 输出日志时间的发生位置，包括类目名，发生的线程，以及在代码中的行数| %l MyClass.main(MyClass.java:129) |
| %L | 显示调用logger的代码行 | %L 129 |
| %m | 显示输出消息 | %m this is a debug message |
| %M | 显示调用logger的方法名 | %M main |
| %n | 当前平台下的换行符 | %n rn(window) n(linux) |
| %p | 显示当前日志的优先级 | %p INFO |
| %r | 显示从程序启动时到记录该条日志时已经经历的毫秒数 | %r 1215|
| %t | 输出产生该日志的事件的线程名 | %t MyClass |
| %x | 按NDC(Nested Diagnostic Context,线程堆栈) 顺序输出日志 |   |
| %X | 按MDC(mapped Diagnostic Context,线程映射表)输出日志。通常用于多客户端连接同一台服务器，方便服务器区分是哪个客户端留下来的日志| %X{5} 记录代号为5的客户端的日志 |
| %% | 显示一个百分号 | %%  % |


log4j配置文件中日志格式的设定:
	log
	log4j.appender.logfile.layout.ConversionPattern=%p %d [%c] [%t] - %m%n


match => {"message" => ["%{DATA:loglevel} %{TIME:logtime} \[%{DATA:logspacename}\] \[%{DATA:logthreadname}\] - %{DATA:logcontent}"]}

                         %{DATA:loglevel} %{TIMESTAMP_ISO8601:logtime} \[%{DATA:logspacename}\] \[%{DATA:logthreadname}\] - %{GREEDYDATA:logcontent}


\{\"%{DATA:returnkey}\":\"%{DATA:returnvalue}\",\"%{DATA:methodkey}\":\"%{DATA:methodname}\",\"%{DATA:argumentskey}\":\"%{DATA:arguments}\",\"%{DATA:classnamekey}\":\"%{DATA:classnamevalue}\",\"%{DATA:elapsedtimekey}\":%{NUMBER:elapsedtime}\}


\{\"%{DATA}\":\"%{DATA:returnvalue}\",\"%{DATA}\":\"%{DATA:methodname}\",\"%{DATA}\":\"%{DATA:arguments}\",\"%{DATA}\":\"%{DATA:classnamevalue}\",\"%{DATA}\":%{NUMBER:elapsedtime}\}


if [loglevel] == "INFO" {
            "logcontent" => ["\{\"%{DATA}\":\"%{DATA:returnvalue}\",\"%{DATA}\":\"%{DATA:methodname}\",\"%{DATA}\":\"%{DATA:arguments}\",\"%{DATA}\":\"%{DATA:classname}\",\"{DATA}\":%{NUMBER:elapsedtime}"]}
        overwrite => ["logcontent"]
        }


if ["loglevel"] == "INFO" {
    grok {
        "match" => {
            "logcontent" => ["\{\"%{DATA}\":\"%{DATA:returnvalue}\",\"%{DATA}\":\"%{DATA:methodname}\",\"%{DATA}\":\"%{DATA:arguments}\",\"%{DATA}\":\"%{DATA:classname}\",\"{DATA}\":%{NUMBER:elapsedtime}\}"]
        remove_field => ["logcontent"]
    }
    }
    }



filter {
    if [type] == "omsweb" {
        if [loglevel] == "INFO" {
            grok {
                match => {
                    "logcontent" => ["\{\"%{DATA}\":\"%{DATA:returnvalue}\",\"%{DATA}\":\"%{DATA:methodname}\",\"%{DATA}\":\"%{DATA:arguments}\",\"%{DATA}\":\"%{DATA:classname}\",\"{DATA}\":%{NUMBER:elapsedtime}\}"]
                remove_field => ["logcontent"]
                }
            }
        }
    }




}



log
DEBUG 2017-09-13 16:51:49,108 [tdh.platform.ofc.cache.RedisCache] [http-nio-8080-exec-8] - Executing function tdh.platform.ofc.cache.RedisCache$3@599c1487
INFO 2017-09-13 16:51:49,108 [tdh.thunder.logging.api.InvocationLogInterceptor] [http-nio-8080-exec-8] - {"returnValue":"泉州天地汇物流发展有限公司","methodName":"queryCenter","arguments":"1131866","className":"tdh.platform.ofc.biz.facade.fin.TargetFacadeImpl@8cd8e6b","elapsedTime":0}
DEBUG 2017-09-13 16:51:49,108 [tdh.platform.ofc.cache.RedisCache] [http-nio-8080-exec-8] - Executing function tdh.platform.ofc.cache.RedisCache$3@5b353d70
INFO 2017-09-13 16:51:49,108 [tdh.thunder.logging.api.InvocationLogInterceptor] [http-nio-8080-exec-8] - {"returnValue":"泉州天地汇物流发展有限公司","methodName":"queryCenter","arguments":"1131866","className":"tdh.platform.ofc.biz.facade.fin.TargetFacadeImpl@8cd8e6b","elapsedTime":0}
INFO 2017-09-13 16:51:49,108 [tdh.thunder.logging.api.InvocationLogInterceptor] [http-nio-8080-exec-8] - {"returnValue":"*(MASKED_VALUE)*","methodName":"selectOrderTargetMappList","arguments":"StatementCriteria [targetUuid=3122817, orderUuid=null, orderNo=, beginDate=2017-06-27, endDate=2017-08-25, orderTypeCd=, initTypePage=true, targetRoleCd=CUST, orderTargetUuids=null, uuids=null]","className":"tdh.platform.ofc.biz.facade.fin.OrderTargetMappingFacadeImpl@68e3ed8b","elapsedTime":154}
INFO 2017-09-13 16:52:01,393 [tdh.thunder.logging.api.InvocationLogInterceptor] [http-nio-8080-exec-4] - {"returnValue":"*(MASKED_VALUE)*","methodName":"listRoles","arguments":"560849","className":"tdh.platform.ofc.biz.acm.impl.SubjectAdminBizImpl@1593d325","elapsedTime":1}
INFO 2017-09-13 16:52:01,393 [tdh.thunder.logging.api.InvocationLogInterceptor] [http-nio-8080-exec-4] - {"returnValue":"*(MASKED_VALUE)*","methodName":"listRoles","arguments":"560849","className":"tdh.platform.ofc.biz.facade.acm.SubjectAdminFacadeImpl@47969c1e","elapsedTime":1}
INFO 2017-09-13 16:52:01,395 [tdh.platform.ofc.web.controller.fin.StatementBaseController] [http-nio-8080-exec-4] - 账单初步筛选列表
INFO 2017-09-13 16:52:01,396 [tdh.thunder.dac.interceptor.SqlStatementInterceptor] [http-nio-8080-exec-4] - SqlStatement(added filtering field):
SELECT count(1) FROM fin_order_target_mapping chargetarget LEFT JOIN `fin_state_order_mapping` stateordermapp ON (stateordermapp.`ORDER_TARGET_UUID` = chargetarget.`UUID` AND stateordermapp.`DELETED` = 0 AND stateordermapp.`STAT_CD` != 'INVA') LEFT JOIN `fin_statement` statement ON (statement.`UUID` = stateordermapp.`STATE_UUID` AND statement.`DELETED` = 0 AND statement.`STATUS_CD` != 'INVA') LEFT JOIN fin_target target on target.TARGET_UUID = chargetarget.TARGET_UUID WHERE chargetarget.PROFILE_ID = 1131866 AND  chargetarget.DELETED = 0 and chargetarget.TARGET_UUID = ? and chargetarget.TARGET_ROLE_CD = ? and DATE(chargetarget.REGISTER_DATETIME) >= ? and DATE(chargetarget.REGISTER_DATETIME) <= ? and chargetarget.ACCOUNT_STATUS_CD = ?