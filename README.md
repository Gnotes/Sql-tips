# Sql-tips

`汇集学习过程中比较实用的sql技巧`  

## Mysql

- `数据表自连接查询`  
[Can't specify target table for update in FROM clause](http://stackoverflow.com/questions/45494/mysql-error-1093-cant-specify-target-table-for-update-in-from-clause)

```sql
update user_accounts as a inner join user_accounts as b   -- 使用inner join 链接查询
  on a.id = b.id set a.status = 0  
  where b.update_at <= DATE_ADD(NOW(),INTERVAL -2 DAY) 
```

- `触发器before执行`   
[Can't update table in stored function/trigger because it is already used by statement which invoked this stored function/trigger](http://stackoverflow.com/questions/15300673/mysql-error-cant-update-table-in-stored-function-trigger-because-it-is-already)

```sql
DELIMITER $
create trigger tri_check_index before update on message                   -- 在更新之前(before)触发
for each row
begin 
  if(old.status != 1 && new.status = 1) then
    set @cur_check_index = (select max(check_index) from message) + 1;
    set new.check_index = @cur_check_index;                               -- 设置新(new)数据的值
  end if;
end;
$
```

- `创建或更新`

**匹配主键或唯一约束键**，如果存在则执行`Update`，否则`Insert`

```sql
INSERT INTO `Table_name` (`id`,`type`,`create_at`,`update_at`) VALUES (3,2,'2017-05-18 11:06:17','2017-05-18 11:06:17') 
ON DUPLICATE KEY UPDATE `id`=VALUES(`id`), `type`=VALUES(`type`), `update_at`=VALUES(`update_at`);
```

- `sql与(&)运算`

[roles-and-permission](https://stackoverflow.com/questions/333620/best-practice-for-designing-user-roles-and-permission-system)  
[MySql 运算符](http://www.cnblogs.com/emanlee/p/4592337.html)  

示例：ACL(access control)计算，有如下表:

### permission

| bit | name |
| --- | ---- |
| 1 | User-Add |
| 2 | User-Edit |
| 4 | User-Delete |
| 8 | User-View |
| 16  | Blog-Add |
| 32  | Blog-Edit |
| 64  | Blog-Delete |
| 128 | Blog-View |


### user

| id | name | role |
| -- | ---- | ---- |
| 1 | Ketan | 65 |
| 2 | Mehata | 132 |

执行查询：

```sql
SELECT user.role, permission.bit, user.role & permission.bit as '位与运算', permission.name  
   FROM user LEFT JOIN permission ON user.role & permission.bit
 WHERE user.id = 2;
```

| role | bit | 位与运算 | name |
| -- | ---- | ---- | ---- |
| 132 | 4 | 4 | User-Delete |
| 132 | 128 | 128 | Blog-View |

```
原理： 十进制转二进制后进行与运算  
132 -> 1000 0100  
4   -> 0000 0100  
128 -> 1000 0000  

132 & 4 = 4
  1000 0100
& 0000 0100
-----------
  0000 0100

  132 & 128 = 128
  1000 0100
& 1000 0000
-----------
  1000 0000
```
