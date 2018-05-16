---
title: 用PL/Proxy为PostgreSQL配置数据库集群
date: 2015-07-08 15:00:18
categories: 数据库
tags: ['PL/Proxy', 'PostgreSQL']
comments: false
img: /2015/07/08/plproxy/PostgreSQL.jpg
---

最近学习使用 PL/Proxy 和 PostgreSQL 来做数据库集群，以免将来忘记，特将学习笔记记录如下：


## 实验环境为：

Fedora 16（三台虚拟机），Postgres  9.1.7，plproxy-2.5
![](plproxy.png)

## 原理：
以三台机器的集群为例子，看看PostgreSQL集群的架构是什么。

proxy节点：proxy节点实际上也是一个PostgreSQL数据库 节点，但是所有数据均不存放到proxy节点上，主要做三件事情：

1. 接受用户的sql查询；
2. 分析用户的sql查询并转换成集群上执行的SQL语句；
3. 合并集群执行sql的结果，然后返回给用户。

说白了，就是把用户的sql语句交给database0，database1去执行，然后合并执行结果返回给用户。

database0 节点和 database1节点：
就是普通的数据库节点，接收proxy节点的sql查询请求并返回结果给proxy节点，是真正存放数据的节点。

## 步骤：
1. 在Fedora 16上安装postgresql数据库：yum install postgresql-server.x86_64
，安装后会创建一个postgres的用户，用户目录在/var/lib/pgsql，需要切换到postgres用户执行initdb命令初始化数据库，可以查看用户目录`startpg.sh`的脚本，可以修改数据库data数据文件位置，执行该脚本便可以完成数据库服务的启动。

2. 从官网下载plproxy安装包，解压后查看其中的README文本，可以大概知道怎么安装，不过先别急，你此时按照README所述执行make 和make install命令可能出错，因为经过我的摸索，在fedora下安装这个需要依赖一些其他软件包。当然，按照参考资料中直接在ubantu下从软件仓库安装postgresql-X.X-plproxy可能例外，不需这么麻烦，但是在fedora yum的仓库中没有，所以需要从官网下载独立安装包。

3. 安装plproxy-2.5之前，输入pg_config命令查看一下postgresql的一些配置，我的如下：
```
BINDIR = /usr/bin
DOCDIR = /usr/share/doc/pgsql
HTMLDIR = /usr/share/doc/pgsql
INCLUDEDIR = /usr/include
PKGINCLUDEDIR = /usr/include/pgsql
INCLUDEDIR-SERVER = /usr/include/pgsql/server
LIBDIR = /usr/lib64
PKGLIBDIR = /usr/lib64/pgsql
LOCALEDIR = /usr/share/locale
MANDIR = /usr/share/man
SHAREDIR = /usr/share/pgsql
SYSCONFDIR = /etc
PGXS = /usr/lib64/pgsql/pgxs/src/makefiles/pgxs.mk
......
```

用`echo $PATH`查看一下，path中是否包含BINDIR，没有的话需要加上哦。
然后需要安装postgresql-devel.x86_64 ，你可以阅读plproxy解压目录下的makefile文件，能大致知道需要一些什么东西。postgresql-devel.x86_64 便提供了配置中 PGXS 所指向的文件，然后还需要安装 flex 和BISON 这两个，然后到plproxy-2.5目录下便可执行 make 和 make install命令了，在/usr/lib64/pgsql/目录中有了plproxy.so这个文件就表示你安装成功了。

4. 安装就绪了就开始配置吧，在`$SHAREDIR(/usr/share/pgsql)`目录下有个安装plproxy安装成功后生成的extension文件夹，该文件夹中有个plproxy--2.5.0.sql脚本，你可以先阅读看看，知道它要干什么，然后在我们的proxy数据节点中执行该脚本。
```
#sudo -u postgres /usr/bin/psql –f /usr/share/pgsql/contrib/extension/plproxy.sql plproxy（数据库名）
```
（对了，还需要修改数据库允许连接的配置，在data文件夹下的pg_hba.conf和postgresql.conf两个文件，修改监听地址和IP认证，具体做法不赘述，自己可以搜。）

5. 在三台机器上分别创建三个数据库，分别是proxy,database0,database1，我用的创建工具是pgadmin3图形界面工具，也推荐使用，用命令行也可以，我嫌麻烦。
6. 在pgadmin3中选择proxy数据库，打开query tool，输入如下语句并执行
create schema plproxy
7. 在pgadmin3中选择proxy数据库的plproxy模式（刚创建的），打开query tool ，输入如下建立3个函数的语句并执行。

- (1)建立plproxy.get_cluster_config函数：
```
CREATE OR REPLACE FUNCTION plproxy.get_cluster_config(IN cluster_name text, OUT key text, OUT val text)
  RETURNS SETOF record AS
$BODY$
BEGIN
key := 'statement_timeout';--就是给 key 变量赋值,赋的值为’statement_timeout’
val := 60;--就是给 val 变量赋值,赋的值为 60
RETURN NEXT;
RETURN;
END;
$BODY$
  LANGUAGE plpgsql VOLATILE
  COST 100
  ROWS 1000;
ALTER FUNCTION plproxy.get_cluster_config(text)
  OWNER TO postgres;
```

- (2)建立plproxy.get_cluster_partitions函数：
```
CREATE OR REPLACE FUNCTION plproxy.get_cluster_partitions(cluster_name text)
  RETURNS SETOF text AS
$BODY$
BEGIN
IF cluster_name = 'testcluster' THEN --cluster_name 是群集的名字
RETURN NEXT 'dbname=database0 host=127.0.0.1'; --数据库节点的数据库名和 IP 地址
RETURN NEXT 'dbname=database1 host=127.0.0.1'; --数据库节点的数据库名和 IP 地址
RETURN;
END IF;
RAISE EXCEPTION 'Unknown cluster'; --如果群集名不存在,抛出异常,这个是在数据库内部处理的,最终会写入日志中。‘Unknown cluster’是报错信息
END;
$BODY$
  LANGUAGE plpgsql VOLATILE
  COST 100
  ROWS 1000;
ALTER FUNCTION plproxy.get_cluster_partitions(text)
  OWNER TO postgres;
```

- (3)建立 plproxy.get_cluster_version函数：
```
CREATE OR REPLACE FUNCTION plproxy.get_cluster_version(cluster_name text)
  RETURNS integer AS
$BODY$
BEGIN
IF cluster_name = 'testcluster' THEN
RETURN 1;
END IF;
RAISE EXCEPTION 'Unknown cluster';
END;
$BODY$
  LANGUAGE plpgsql VOLATILE
  COST 100;
ALTER FUNCTION plproxy.get_cluster_version(text)
  OWNER TO postgres;
```

8. 在pgadmin3中选择proxy数据库的public模式（默认就有的），打开query tool ，输入如下建立3个函数的语句并执行

- (1)建立public.ddlexec(sql_request text)函数：
```
CREATE OR REPLACE FUNCTION ddlexec(query text)
  RETURNS SETOF integer AS
$BODY$
CLUSTER 'testcluster';
RUN ON ALL;
$BODY$
  LANGUAGE plproxy VOLATILE
  COST 100
  ROWS 1000;
ALTER FUNCTION ddlexec(text)
  OWNER TO postgres;
```

- (2)建立public.dmlexec(sql_request text)函数：
```
CREATE OR REPLACE FUNCTION dmlexec(query text)
  RETURNS SETOF integer AS
$BODY$
CLUSTER 'testcluster';
RUN ON ANY;
$BODY$
  LANGUAGE plproxy VOLATILE
  COST 100
  ROWS 1000;
ALTER FUNCTION dmlexec(text)
  OWNER TO postgres;
```

- (3)建立public.dqlexec(sql_request text)函数：
```
CREATE OR REPLACE FUNCTION dqlexec(query text)
  RETURNS SETOF record AS
$BODY$
CLUSTER 'testcluster';
RUN ON ALL;
$BODY$
  LANGUAGE plproxy VOLATILE
  COST 100
  ROWS 1000;
ALTER FUNCTION dqlexec(text)
  OWNER TO postgres;
```

9. 在pgadmin3中选择database0 和database1数据库的public模式（默认就有的），打开query tool ，输入如下建立3个函数的语句并执行

- (1)建立public.ddlexec(sql_request text)函数：
```
CREATE OR REPLACE FUNCTION ddlexec(query text)
RETURNS integer AS
$BODY$
declare
ret integer;
begin
execute query;
return 1;
end;
$BODY$
LANGUAGE 'plpgsql' VOLATILE
COST 100;
```

- (2)建立public.dmlexec(sql_request text)函数：
```
CREATE OR REPLACE FUNCTION dmlexec(query text)
RETURNS integer AS
$BODY$
declare
ret integer;
begin
execute query;
return 1;
end;
$BODY$
LANGUAGE 'plpgsql' VOLATILE
COST 100;
```

- (3)建立public.dqlexec(sql_request text)函数：
```
CREATE OR REPLACE FUNCTION dqlexec(query text)
RETURNS SETOF record AS
$BODY$
declare
ret record;
begin
for ret in execute query loop
return next ret;
end loop;
return;
end;
$BODY$
LANGUAGE 'plpgsql' VOLATILE
COST 100
ROWS 1000;
```

10. 到此为止，集群就创建成功了，现在就可以通过proxy这个节点对 database0 和database1 进行建表、插入数据、查询数据的操作了
- (1)创建表usertable：

在pgadmin3中选择proxy数据库，打开query tool ，输入如下建立usertable的语句并执行
```
select ddlexec('create table usertable(id integer)');
```

上述语句执行完，在database0和database1数据库的public模式中就应该有usertable这个表了，如果没有这个表，那就是前面某一步出错了啦。

- (2)在usertable表中插入数据：

在pgadmin3中选择proxy数据库，打开query tool ，输入如下建立usertable的语句并执行
```
select dmlexec('insert into usertable values(0)');
select dmlexec('insert into usertable values(1)');
select dmlexec('insert into usertable values(2)');
select dmlexec('insert into usertable values(3)');
select dmlexec('insert into usertable values(4)');
select dmlexec('insert into usertable values(5)');
select dmlexec('insert into usertable values(6)');
select dmlexec('insert into usertable values(7)');
select dmlexec('insert into usertable values(8)');
select dmlexec('insert into usertable values(9)');
select dmlexec('insert into usertable values(10)');
```

上述语句执行完之后能看到database0和database1这两个数据库的usertable表中已经有10行数据了啦。

- (3)搜索usertable表中数据：

在pgadmin3中选择proxy数据库，打开query tool ，输入如下建立usertable的语句并执行
```
select * from dqlexec('select * from usertable ') as (id integer);
```

获得结果。