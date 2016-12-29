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