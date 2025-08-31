# 第十二章：创建触发器

![](img/chapterart.png)

在本章中，你将创建触发器，数据库对象，它们会在一行被插入、更新或删除之前或之后自动*触发*，执行你定义的功能。每个触发器都与一个表关联。

触发器最常用于跟踪对表的更改，或在数据被保存到数据库之前增强数据质量。

像函数和存储过程一样，触发器会保存在你创建它们的数据库中。

## 数据审计触发器

你将首先使用触发器来跟踪对数据库表的更改，方法是创建一个第二个*审计表*，记录哪个用户更改了哪条数据，并保存更改的日期和时间。

请看一下公司`accounting`数据库中的`payable`表。

```
payable_id  company            amount   service
----------- -------           -------   ------------------------
     1      Acme HVAC          123.32   Repair of Air Conditioner
     2      Initech Printers  1459.00   Printer Repair
     3      Hooli Cleaning     398.55   Janitorial Services
```

为了创建一个审计表来跟踪对`payable`表进行的任何更改，可以输入如下内容：

```
create table payable_audit
  (
    audit_datetime  datetime,
    audit_user      varchar(100),
    audit_change    varchar(500)
  );
```

你将创建触发器，以便在对`payable`表进行更改时，将更改的记录保存到`payable_audit`表中。你将保存更改的日期和时间到`audit_datetime`列；保存进行更改的用户到`audit_user`列；以及将更改内容的文本描述保存到`audit_change`列。

触发器可以设置在行数据更改之前或之后触发。你将创建的第一组触发器是*after*触发器。你将设置三个触发器，在`payable`表的数据发生更改后触发。

### 插入后的触发器

一个*插入后*触发器（在代码中通过后缀`_ai`表示）会在一行被插入后触发。示例 12-1 展示了如何为`payable`表创建一个插入后触发器。

```
use accounting;

drop trigger if exists tr_payable_ai;

delimiter //

❶ create trigger tr_payable_ai
❷ after insert on payable
❸ for each row
begin
❹ insert into payable_audit
  (
    audit_datetime,
    audit_user,
    audit_change
  )
  values
  (
    now(),
 user(),
    concat(
     'New row for payable_id ',
    ❺ new.payable_id,
      '. Company: ',
      new.company,
      '. Amount: ',
      new.amount,
      '. Service: ',
      new.service
    )
  );
end//

delimiter ;
```

示例 12-1：创建一个插入后触发器

首先，你创建触发器并命名为`tr_payable_ai` ❶。接着，你指定`after`关键字来表示触发器应当在何时触发 ❷。在这个例子中，一行将被插入到`payable`表中，*然后*触发器将触发，将审计记录写入`payable_audit`表。

在触发器中，对于每一行 ❸ 被插入到`payable`表中时，MySQL 会执行`begin`和`end`语句之间的代码。所有触发器都会包含`for each row`语法。

你通过一个`insert`语句插入一行数据到`payable_audit`表中，该语句调用了三个函数：`now()`用于获取当前日期和时间；`user()`用于获取插入该行的用户的用户名；`concat()`用于构建一个描述插入到`payable`表中数据的字符串 ❹。

在编写触发器时，你使用`new`关键字来访问插入到表中的新值 ❺。例如，你可以通过引用`new.payable_id`来获取新的`payable_id`值，通过引用`new.company`来获取新的`company`值。

现在你已经设置了触发器，尝试向`payable`表中插入一行数据，看看新的行是否会自动在`payable_audit`表中被跟踪：

```
insert into payable
  (
    payable_id,
    company,
    amount,
    service
  )
values
  (
 4,
    'Sirius Painting',
    451.45,
    'Painting the lobby'
  );

select * from payable_audit;
```

结果显示触发器生效了。向 `payable` 表插入新行导致你的 `tr_payable_ai` 触发器被触发，从而向你的 `payable_audit` 审计表插入了一行：

```
audit_datetime       audit_user      audit_change
-------------------  --------------  -----------------------------------------
2024-04-26 10:43:14  rick@localhost  New row for payable_id 4.
                                     Company: Sirius Painting. Amount: 451.45.
                                     Service: Painting the lobby
```

`audit_datetime` 列显示了行被插入的日期和时间。`audit_user` 列显示了插入该行的用户的用户名和主机（*主机* 是 MySQL 数据库所在的服务器）。`audit_change` 列包含了使用 `concat()` 函数构建的新增行的描述。

### 删除后触发器

现在你将编写一个 *删除后* 触发器（在代码中以 `_ad` 后缀表示），它会将从 `payable` 表中删除的任何行记录到 `payable_audit` 表中（列表 12-2）。

```
use accounting;

drop trigger if exists tr_payable_ad;

delimiter //

create trigger **tr_payable_ad**
 **after delete on payable**
  for each row
begin
  insert into payable_audit
    (
      audit_date,
      audit_user,
      audit_change
    )
  values
    (
      now(),
      user(),
      concat(
       'Deleted row for payable_id ',
     ❶ old.payable_id,
       '. Company: ',
       old.company,
       '. Amount: ',
       old.amount,
       '. Service: ',
       old.service
 )
  );
end//

delimiter ;
```

列表 12-2：创建一个删除后触发器

`delete` 触发器看起来与 `insert` 触发器类似，但有一些不同之处；即你使用了 `old` 关键字 ❶ 而不是 `new`。由于该触发器在行被删除时触发，因此列只有 `old` 值。

在你设置好删除触发器后，从 `payable` 表中删除一行，看看删除是否会记录到 `payable_audit` 表中：

```
delete from payable where company = 'Sirius Painting';
```

结果如下：

```
audit_datetime       audit_user      audit_change
-------------------  --------------  -----------------------------------------
2024-04-26 10:43:14  rick@localhost  New row for payable_id 4.
                                     Company: Sirius Painting. Amount: 451.45.
                                     Service: Painting the lobby
2024-04-26 10:47:47  rick@localhost  Deleted row for payable_id 4.
                                     Company: Sirius Painting. Amount: 451.45.
                                     Service: Painting the lobby
```

触发器工作了！`payable_audit` 表仍然包含你插入到 `payable` 表中的行，但你也有一行记录了删除操作。

无论是插入行还是删除行，你都将更改记录到同一个 `payable_audit` 表中。你将文本 `New row` 或 `Deleted row` 作为 `audit_change` 列值的一部分，以明确所执行的操作。

### 更新后触发器

要编写一个 *更新后* 触发器（`_au`），它将记录在 `payable` 表中更新的任何行到 `payable_audit` 表中，请在 列表 12-3 中输入代码。

```
use accounting;

drop trigger if exists tr_payable_au;

delimiter //

create trigger **tr_payable_au**
 **after update on payable**
  for each row
begin
  ❶ set @change_msg =
       concat(
              'Updated row for payable_id ',
 old.payable_id
       );

❷ if (old.company != new.company) then
    set @change_msg =
         concat(
              @change_msg,
              '. Company changed from ',
              old.company,
              ' to ',
              new.company
         );
  end if;

  if (old.amount != new.amount) then
    set @change_msg =
         concat(
              @change_msg,
              '. Amount changed from ',
              old.amount,
              ' to ',
              new.amount
         );
  end if;

  if (old.service != new.service) then
    set @change_msg =
         concat(
              @change_msg,
              '. Service changed from ',
              old.service,
              ' to ',
              new.service
         );
  end if;

❸ insert into payable_audit
       (
      audit_datetime,
      audit_user,
      audit_change
    )
  values
    (
       now(),
       user(),
       @change_msg
  );

end//

delimiter ;
```

列表 12-3：创建一个更新后的触发器

你声明此触发器在 `payable` 表更新后触发。当你更新表中的一行时，你可以更新其中一个或多个列。你设计的更新后触发器仅显示 `payable` 表中发生变化的列值。例如，如果你没有更改 `service` 列，你就不会在 `payable_audit` 表中包含任何关于 `service` 列的文本。

你创建了一个名为 `@change_msg` 的用户变量 ❶（用于 *更改消息*），它用来构建一个包含每个更新列的列表的字符串。你检查 `payable` 表中的每个列是否发生了变化。如果旧的 `company` 列值与新的 `company` 列值不同，你会将文本 `Company changed from` `old value` `to` `new value` 添加到 `@change_msg` 变量 ❷。然后，你对 `amount` 和 `service` 列做同样的处理，调整消息文本。完成后，`@change_msg` 的值被插入到 `payable_audit` 表的 `audit_change` 列中 ❸。

设置好更新后触发器后，看看当用户更新 `payable` 表中的一行时会发生什么：

```
update payable
set    amount = 100000,
       company = 'House of Larry'
where  payable_id = 3;
```

`payable_audit`表中的前两行数据仍然出现在结果中，同时还新增了一行，记录了`update`语句的操作：

```
audit_datetime       audit_user      audit_change
-------------------  --------------  -----------------------------------------
2024-04-26 10:43:14  rick@localhost  New row for payable_id 4.
                                     Company: Sirius Painting. Amount: 451.45.
                                     Service: Painting the lobby
2024-04-26 10:47:47  rick@localhost  Deleted row for payable_id 4.
                                     Company: Sirius Painting. Amount: 451.45.
                                     Service: Painting the lobby
2024-04-26 10:49:20  larry@localhost Updated row for payable_id 3\. Company
                                     changed from Hooli Cleaning to House of
                                     Larry. Amount changed from 4398.55 to
                                     100000.00
```

看起来有一个名为`larry@localhost`的用户更新了一行数据，将`amount`更改为$100,000，并将将支付对象的`company`更改为`House of Larry`。嗯……

## 影响数据的触发器

你还可以编写触发器，在行被更改之前触发，以更改写入表中的数据或防止插入或删除行。这有助于在将数据保存到数据库之前提高数据的质量。

在`bank`数据库中创建一个`credit`表，用来存储客户及其信用分数：

```
create table credit
  (
    customer_id    int,
    customer_name  varchar(100),
    credit_score   int
  );
```

和后触发器一样，插入前、删除前和更新前的三个触发器会在行被插入、删除或更新之前触发。

### 插入前触发器

*插入前*触发器（`_bi`）在插入新行之前触发。列表 12-4 展示了如何编写一个插入前触发器，以确保不会有低于 300 或高于 850 的信用分数（即最低信用分数和最高信用分数）被插入到`credit`表中。

```
use bank;

delimiter //

❶ create trigger tr_credit_bi
❷ before insert on credit
  for each row
begin
❸ if (new.credit_score < 300) then
    set new.credit_score = 300;
  end if;

❹ if (new.credit_score > 850) then
    set new.credit_score = 850;
  end if;

end//

delimiter ;
```

列表 12-4：创建插入前触发器

首先，你将触发器命名为`tr_credit_bi` ❶，并定义为`插入前`触发器 ❷，这样它将在行插入到`credit`表之前触发。由于这是一个插入触发器，你可以利用`new`关键字检查即将插入`credit`表中的`new.credit_score`值是否小于 300。如果是这样，你将其设置为`300` ❸。对于超过 850 的信用分数，你也会做类似的检查，将其值更改为`850` ❹。

向`credit`表插入一些数据，看看你的触发器有什么效果：

```
insert into credit
  (
    customer_id,
 customer_name,
    credit_score
  )
values
  (1, 'Milton Megabucks',   987),
  (2, 'Patty Po',           145),
  (3, 'Vinny Middle-Class', 702);
```

现在查看`credit`表中的数据：

```
select * from credit;
```

结果是：

```
customer_id  customer_name       credit_score
-----------  ------------------  ------------
     1       Milton Megabucks        850
     2       Patty Po                300
     3       Vinny Middle-Class      702
```

你的触发器起作用了。它将 Milton Megabucks 的信用分数从`987`改为`850`，并将 Patti Po 的信用分数从`145`改为`300`，这两个值在它们被插入到`credit`表之前就已发生变化。

### 更新前触发器

*更新前*触发器（`_bu`）在表更新之前触发。你已经编写了一个触发器，防止`insert`语句将信用分数设置为低于 300 或高于 850 的值，但有可能`update`语句也会将信用分数更新到这个范围之外。列表 12-5 展示了如何创建一个`before` `update`触发器来解决这个问题。

```
use bank;

delimiter //

create trigger **tr_credit_bu**
 **before update on credit**
  for each row
begin
  if (new.credit_score < 300) then
    set new.credit_score = 300;
  end if;

  if (new.credit_score > 850) then
    set new.credit_score = 850;
  end if;

end//

delimiter ;
```

列表 12-5：创建更新前触发器

更新一行以测试你的触发器：

```
update credit
set    credit_score = 1111
where  customer_id = 3;
```

现在查看`credit`表中的数据：

```
select * from credit;
```

结果是：

```
customer_id  customer_name       credit_score
-----------  ------------------  ------------
     1       Milton Megabucks        850
     2       Patty Po                300
     3       Vinny Middle-Class      850
```

它起作用了。触发器不允许你将`Vinny Middle-Class`的信用分数更新为`1111`。相反，在更新该行数据之前，它将值设置为`850`。

### 删除前触发器

最后，一个*删除前*触发器（`_bd`）会在行被从表中删除之前触发。你可以使用删除前触发器作为检查，在允许删除行之前进行验证。

假设你的银行经理要求你编写一个触发器，防止用户删除`credit`表中信用评分超过 750 的客户。你可以通过编写一个删除前触发器来实现这一点，如列表 12-6 所示。

```
use bank;

delimiter //

create trigger **tr_credit_bd**
 **before delete on credit**
  for each row
begin
❶ if (old.credit_score > 750) then
    signal sqlstate '45000'
    set message_text = 'Cannot delete scores over 750';
  end if;
end//

delimiter ;
```

列表 12-6：创建一个删除前触发器

如果你即将删除的行的信用评分超过 750，触发器会返回错误❶。你使用`signal`语句来处理错误返回，后面跟着`sqlstate`关键字和代码。*sqlstate 代码*是一个五个字符的代码，用于标识特定的错误或警告。由于你正在创建自己的错误，你使用`45000`，它代表一个用户定义的错误。然后，你定义`message_text`来显示你的错误信息。

通过从`credit`表中删除一些行来测试你的触发器：

```
delete from credit where customer_id = 1;
```

由于客户`1`的信用评分为 850，因此结果是：

```
Error Code: 1644\. Cannot delete scores over 750
```

你的触发器起作用了。它阻止了删除该行，因为信用评分超过了 750。

现在删除客户`2`的行，他的信用评分为 300：

```
delete from credit where customer_id = 2;
```

你会收到一条消息，告知你该行已被删除：

```
1 row(s) affected.
```

你的触发器按预期工作。它允许你删除客户`2`的行，因为他们的信用评分不超过 750，但阻止你删除客户`1`的行，因为他们的信用评分超过了 750。

## 总结

在本章中，你创建了触发器，这些触发器会自动触发并执行你定义的任务。你了解了前触发器和后触发器之间的区别，以及每种触发器的三种类型。你使用触发器来跟踪表的变化，防止特定行被删除，并控制允许值的范围。

在下一章中，你将学习如何使用 MySQL 事件来调度任务。
