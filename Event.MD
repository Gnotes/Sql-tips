# Event 事件

MySql 可以通过事件调用 `定时任务 Schedule` 或 `存储过程 Procedule`.  

## Schedule 定时任务

### 查看定时任务是否开启

```sql
SELECT @@event_scheduler; 
-- @@event_scheduler
-- ON / OFF

-- 或者

SHOW PROCESSLIST; 
-- ID   User            HOST      DB    COMMAND TIME  STATE                         INFO
-- 3499 event_scheduler localhost NULL  Daemon  52    Waiting for next activation   NULL

-- 查看所有事件

SHOW EVENTS;
```

### 开启任务

#### 临时开启，重启后失效

```sql
SET GLOBAL event_scheduler = ON;
-- SET GLOBAL event_scheduler = 1;   —- 0 代表关闭

SET @@global.event_scheduler = ON;
-- SET@@global.event_scheduler = 1;
```

#### 永久开启

在MySQL的 `my.conf` 中添加如下命令，并 **重启**

- [Linux]
  位置：*[my.conf]* 目录一般在 `/etc/my.conf` `/etc/mysql/my.conf` `/usr/etc/my.conf` `~/.my.conf`  
- [Windows]
  位置：安装目录的`my.ini` `my-default.ini`

```sql
event_scheduler=ON
```

##### 编译安装的时候指定参数

```sql
mysqld –event_scheduler=ON
```

### 启动已关闭的事件

```sql
ALTER EVENT event_name ENABLE;
```

### 关闭已启动的事件

```sql
ALTER EVENT event_name DISABLE;
```

### 删除事件

```sql
DROP EVENT [IF EXISTS] event_name;
```

### 创建定时任务

语法：

```sql
CREATE [DEFINER = { user| CURRENT_USER}]
EVENT [IFNOT EXISTS] event_name
ON SCHEDULE schedule 
[ONCOMPLETION [NOT] PRESERVE] 
[ENABLE | DISABLE | DISABLEON SLAVE] 
[COMMENT 'some comments'] 
DO event_body;
```

**schedule** 可配置项:  
- `AT` timestamp [+ INTERVAL interval]
- `EVERY` interval [STARTS timestamp[+ INTERVAL interval] ...] [ENDS timestamp[+ INTERVAL interval] ...]

**interval** 可配置项:  
quantity {YEAR| QUARTER | MONTH| DAY| HOUR| MINUTE| WEEK | SECOND| YEAR_MONTH | DAY_HOUR | DAY_MINUTE | DAY_SECOND | HOUR_MINUTE | HOUR_SECOND | MINUTE_SECOND}

*参数详细说明：*

- DEFINER  
  定义事件执行的时候检查权限的用户。
- ON SCHEDULE   
  schedule: 定义执行的时间和时间间隔。
- ON COMPLETION [NOT] PRESERVE  
  定义事件是一次执行还是永久执行，默认为一次执行，即NOT PRESERVE。
- AT   
  表示指定一个时间只执行一次。
- EVERY   
  周期性计划，可指定计划开始时间 STARTS 和结束时间 ENDS,执行具体时间周期可以为：YEAR, MONTH, WEEK, DAY, HOUR, MINUTE, SECOND。
- COMPLETION   
  计划任务完毕后，该计划任务对象是否还在数据库中保留。默认不保留  
- ENABLE | DISABLE | DISABLE ON SLAVE  
  定义事件创建以后是开启还是关闭，以及在从上关闭。如果是从服务器自动同步创建事件的语句的话，会自动加上DISABLE ON SLAVE。
- COMMENT   
  定义事件的注释

EX:

```sql
CREATE EVENT IF NOT EXISTS EVENT_UPDATE_EXPIRE_ORDER 
ON SCHEDULE EVERY 1 MINUTE 
ON COMPLETION PRESERVE 
DO UPDATE db.order SET db.order.status = 7 WHERE db.order.status = 1 and db.order.payment_expire_at <= CURRENT_TIMESTAMP;
```

EX:

5天后开启每天定时清空t_order表，一个月后停止执行

```sql
CREATEEVENT event_truncate_table
ON SCHEDULE EVERY 1 DAY
STARTS CURRENT_TIMESTAMP + INTERVAL 5 DAY
ENDS CURRENT_TIMESTAMP + INTERVAL 1 MONTH
DO TRUNCATE TABLE db.t_order;
```

### 修改定时任务

```sql
ALTER EVENT [DEFINER = { user| CURRENT_USER}] event_name
[ONSCHEDULE schedule]
[RENAMETO new_event_name]
[ONCOMPLETION [NOT] PRESERVE]
[ENABLE | DISABLE | SLAVESIDE_DISABLED]
[COMMENT 'comment'] 
[DO sql_statement];
```
 

### 调用存储过程

创建存储过程：

```sql
DROP PROCEDURE IF EXISTS procedule_name;
DELIMITER
CREATE PROCEDURE procedule_name()
BEGIN
DECLARE 1_id INTDEFAULT 1;
DECLARE 1_id2 INTDEFAULT 0;
DECLARE error_status INTDEFAULT 0;
DECLARE datas CURSORFOR SELECT id FROM t_order;
DECLARE CONTINUE HANDLER FOR NOT FOUND SET error_status=1;
OPEN datas;
FETCH datas INTO1_id;
REPEAT
SET 1_id2=1_id2+2;
UPDATE t_order SET id2=1_id2 WHERE id=1_id;
FETCH datas INT O1_id;
UNTIL error_status
ENDREPEAT;
CLOSE datas;
END
```

调用：

```sql
CREATE EVENT call_procedule_by_event 
ON SCHEDULE EVERY 1 DAY
STARTS '2012-08-22 00:00:00'
ENDS '2012-08-22 00:00:00'+INTERVAL 40 DAY
ON COMPLETION PRESERVE 
DO
CALL procedule_name();
```


- [参考文档](http://blog.csdn.net/linybo/article/details/39002055)
