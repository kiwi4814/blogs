---
title: Docker安装Oracle 11G
date: 2024-01-06T16:46:31.000
slug: /docker-oracle
draft: false
tags:
  - Docker
  - 软件
categories:
  - Docker
series:
  - Docker
enableTOC: false
---



现在因为业务需求我要在我的macos系统上（inter芯片）安装一个oracle数据库，对方指定了版本为oracle_11g，并提供了创建表空间和用户的脚本以及相应的dmp文件。

创建表空间的SQL文件（create_tablespaces.sql）如下：

```sql
variable v_count_data number;
variable v_count_index number;
variable v_path varchar2(100);
variable v_dync_sql1 varchar2(200);
variable v_dync_sql2 varchar2(200);

begin
     select substr(name,1,instr(name,'SYSTEM01.DBF',1)-1) into :v_path from v$datafile where file#=1;

     select count(*) into :v_count_data from dba_tablespaces where  tablespace_name = 'ORCL';
     :v_dync_sql1 := 'CREATE TABLESPACE orcl  DATAFILE '||chr(39)||:v_path||'orcl.dbf'||chr(39)||' SIZE 100M autoextend on next 1024k maxsize unlimited';
       
     if (:v_count_data = 0) then
           execute immediate :v_dync_sql1;
     end if;
end;
/
```

创建用户的SQL文件（create_user.sql）如下：

```sql
variable v_count number;
begin
     select count(*) into :v_count from dba_users where username = 'XF119';
     if (:v_count > 0) then
         execute immediate 'drop user XF119 cascade';
     end if;
         execute immediate 'create user XF119 identified by "1234" default tablespace orcl';
         execute immediate 'grant dba to XF119';
         execute immediate 'grant select on dba_data_files to XF119';
         execute immediate 'grant select on dba_free_space to XF119';
end;
/
```

对方提供的dmp文件的路径是：`/Users/heqifeng/Downloads/xy/20240208/20240208.dmp`

我打算使用docker安装（docker run -d --privileged --name oralce11g -p 1521:1521 iatebes/oracle_11g），并且使用navicat进行连接和使用。我找到一个仓库是 iatebes/oracle_11g，现在我需要从头开始学习oracle数据库的连接和安装，希望您能深度思考并给我提供一些帮助，我的要求是：

1. 给我详细的安装步骤

2. 安装完成后执行的SQL尽量使用navicat来执行，docker容器内不便操作，如果上面的SQL跟navicat有兼容性问题，麻烦帮我改一下

3. dmp文件是在我的本地路径的，所以docker安装是不是要挂载？

4. 记住这次的提问内容，作为我的前提条件，后续的回答中都要记住我的最原始的需求：以能够成功导入dmp文件为最终目标

   

## 安装流程

1. 创建 oracle 的数据目录

   ```
   mkdir ~/oracle_data
   
   mkdir ~/oracle_data/dmp
   ```

2. 复制原始dmp文件到刚创建好的目录中

   ```
   cp /Users/heqifeng/Downloads/xy/20240208/20240208.dmp ~/oracle_data/dmp
   ```

3. 启动docker容器

   ```
   docker run -d \
   --privileged \
   --name oracle11g \
   -p 1521:1521 \
   -v ~/oracle_data:/opt/oracle/oradata \
   -v ~/oracle_data/dmp:/opt/oracle/dmp \
   iatebes/oracle_11g
   ```

4. 测试

   使用Navicat连接系统账户（默认密码 oracle）：`system/oracle `

## 创建用户和空间

在navicat的console中执行

1. 检查表空间是否存在

   ```
   SELECT COUNT(*) FROM dba_tablespaces WHERE tablespace_name = 'ORCL';
   ```

2. 创建表空间（如果表空间不存在）

   ```sql
   CREATE TABLESPACE orcl
   DATAFILE '/opt/oracle/oradata/orcl.dbf'
   SIZE 100M
   AUTOEXTEND ON NEXT 1024K
   MAXSIZE UNLIMITED;
   ```

3. 检查用户是否存在

   ```
   SELECT COUNT(*) FROM dba_users WHERE username = 'XF119';
   ```

4. 删除用户 (如果用户存在)

   ```
   DROP USER XF119 CASCADE;
   ```

5. 创建用户

   ```
   CREATE USER XF119 IDENTIFIED BY "1234" DEFAULT TABLESPACE orcl;
   ```

6. 授予 DBA 权限

   ```
   GRANT dba TO XF119;
   ```

7. 授予数据字典访问权限

   ```
   GRANT SELECT ON dba_data_files TO XF119;
   GRANT SELECT ON dba_free_space TO XF119;
   ```

8. 创建 DIRECTORY 对象（使用 system 用户执行，只需要执行一次）

   ```
   CREATE OR REPLACE DIRECTORY dmpdir AS '/opt/oracle/dmp';
   GRANT READ,WRITE ON DIRECTORY dmpdir TO XF119;
   ```

9. ~~执行导入命令 （使用 XF119 用户执行）~~

   ```
    -- 注意修改  `DUMPFILE` 和 `REMAP_SCHEMA` 参数
      
    impdp XF119/1234@orcl  DIRECTORY=dmpdir DUMPFILE=20240208.dmp   REMAP_SCHEMA=TEST:XF119;
   ```

## 导入数据

其中导入的时候暂时行不通，改为在docker中执行：

1. 进入docker容器内

   ```
   docker exec -it oracle11g bash
   ```

2. 执行命令

   ```
   impdp XF119/1234@orcl DIRECTORY=dmpdir DUMPFILE=20240208.dmp REMAP_SCHEMA=XF119:XF119;
   ```

   报错：`bash: impdp: command not found`

3. 写入环境变量方便后续执行

   ```bash
   # 查找 impdp 命令的路径
   find / -name impdp 2>/dev/null
   # 执行成功后，你会看到 impdp 命令的完整路径。例如，它可能类似于 /opt/oracle/product/11.2.0/dbhome_1/bin/impdp。
   export PATH=$PATH:/opt/oracle/product/11.2.0/dbhome_1/bin
   ```

   - `find /`:  在整个文件系统中搜索。  

   * `-name impdp`:  查找名为 `impdp` 的文件。  
   * `2>/dev/null`:  将错误输出重定向到空设备，防止输出不必要的错误信息。

4. 再次执行命令

   ```
   impdp XF119/1234@orcl DIRECTORY=dmpdir DUMPFILE=20240208.dmp REMAP_SCHEMA=XF119:XF119;
   ```

   新的报错信息：

   ```
   impdp XF119/1234@orcl DIRECTORY=dmpdir DUMPFILE=20240208.dmp REMAP_SCHEMA=XF119:XF119;
   UDI-00013: Message 13 not found; No message file for product=RDBMS, facility=UDI
   UDI-00019: You may need to set ORACLE_HOME to your Oracle software directory
   ```

   这表明 Oracle Data Pump 工具 (impdp) 无法找到 Oracle 客户端的安装路径，需要设置 ORACLE_HOME 环境变量。

5. 再次写入环境变量

   ```
   find / -name sqlplus 2>/dev/null
   ```

   这个命令会搜索 `sqlplus` 这个 Oracle 客户端工具的路径，通常 `ORACLE_HOME` 就是它的上一级目录。 *  如果你找到了 `sqlplus` 的路径，比如是 `/opt/oracle/product/11.2.0/dbhome_1/bin/sqlplus`，那么 `ORACLE_HOME` 通常就是 `/opt/oracle/product/11.2.0/dbhome_1`。

   确定 Oracle 安装路径后，使用以下命令设置 ORACLE_HOME 环境变量（请将 /opt/oracle/product/11.2.0/dbhome_1 替换为你实际的 Oracle 安装路径）：

   ```
   export ORACLE_HOME=/opt/oracle/product/11.2.0/dbhome_1
   ```

   设置 ORACLE_HOME 环境变量后，还需要把 $ORACLE_HOME/bin 路径添加到 PATH 环境变量中。

   ```
   export PATH=$PATH:$ORACLE_HOME/bin
   ```

6. 执行之前的命令

   报错：

   ```
   Import: Release 11.2.0.1.0 - Production on Fri Jan 24 06:22:00 2025
   
   Copyright (c) 1982, 2009, Oracle and/or its affiliates.  All rights reserved.
   
   UDI-12154: operation generated ORACLE error 12154
   ORA-12154: TNS:could not resolve the connect identifier specified
   ```

   分析：

   Docker 环境的 TNS 配置： 在 Docker 容器中，默认可能没有正确配置 tnsnames.ora 文件，或者容器内的网络配置可能不匹配，导致 impdp 无法解析 @orcl 连接字符串。我们直接使用完整的连接字符串，彻底避免TNS。

7. 使用新的命令导入

   ```
   impdp XF119/1234@//localhost:1521/orcl DIRECTORY=dmpdir DUMPFILE=20240208.dmp;
   ```

8. 报错

   ```
   impdp XF119/1234@//localhost:1521/orcl DIRECTORY=dmpdir DUMPFILE=20240208.dmp;
   
   Import: Release 11.2.0.1.0 - Production on Fri Jan 24 06:23:06 2025
   
   Copyright (c) 1982, 2009, Oracle and/or its affiliates.  All rights reserved.
   
   Connected to: Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - 64bit Production
   With the Partitioning, Oracle Label Security, OLAP, Data Mining,
   Oracle Database Vault and Real Application Testing options
   ORA-39001: invalid argument value
   ORA-39000: bad dump file specification
   ORA-39143: dump file "/opt/oracle/dmp/20240208.dmp" may be an original export dump file
   ```

   **问题分析**

   - **exp vs expdp:** exp (export) 是旧版本的导出工具，生成的 dmp 文件格式与 expdp (Data Pump export) 生成的 dmp 文件格式不兼容。
   - **impdp 限制:** impdp 导入工具只能导入由 expdp 导出的 dmp 文件。
   - **ORA-39143:** 错误 ORA-39143 明确指出你正在尝试导入的 dmp 文件可能是一个原始的 export 文件。

   **最终解决方案**

   由于你提供的 dmp 文件是使用 exp 导出的，而你现在使用的是 impdp 工具，所以必须使用 imp 工具进行导入。 我们需要**完全切换回使用imp工具**，并同时解决之前的tns问题。

9. 使用旧的命令

   ```
   imp XF119/1234@//localhost:1521/orcl file=/opt/oracle/dmp/20240208.dmp full=y log=test.log commit=y ignore=y;
   ```

   