# 安装说明

## 安装系统依赖

- emcsites/ahi & Oracle

      sudo yum install git gcc-c++ patch openssl-devel libjpeg-devel  libxslt-devel make which python-devel  readline-devel wv poppler-utils binutils gcc gcc-gfortran make m4 perl tar git perl-ExtUtils-MakeMaker texlive httpd compat-libstdc++-33 compat-libstdc++-33.i686 elfutils-libelf elfutils-libelf-devel glibc glibc.i686 glibc-common glibc-devel glibc-devel.i686 glibc-headers ksh libaio libaio.i686 libaio-devel libaio-devel.i686 libgcc libgcc.i686 libstdc++ libstdc++.i686 libstdc++-devel libXi libXtst sysstat unixODBC unixODBC-devel openssl python-docutils python-pip -y

## 创建用户与组(root用户运行)

- firewall & SELinux

      systemctl stop firewalld
      setenforce 0

- emc用户与组

      groupadd emc;\
      groupadd emcgroup;\
      useradd -g emcgroup emc;\
      passwd emc

- oracle

      groupadd oinstall;\
      groupadd dba;\
      useradd -g oinstall -G dba oracle;\
      passwd oracle

- 修改内核参数

      echo -e "fs.aio-max-nr = 1048576\nfs.file-max = 6815744\nkernel.shmall = 2097152\nkernel.shmmax = 536870912\nkernel.shmmni = 4096\nkernel.sem = 250 32000 100 128\nnet.ipv4.ip_local_port_range = 9000 65500\nnet.core.rmem_default = 262144\nnet.core.rmem_max = 4194304\nnet.core.wmem_default = 262144\nnet.core.wmem_max = 1048586" >/etc/sysctl.conf;\
      sysctl -p


<!-- ----------------

      vi /etc/sysctl.conf

      fs.aio-max-nr = 1048576
      fs.file-max = 6815744
      kernel.shmall = 2097152
      kernel.shmmax = 536870912
      kernel.shmmni = 4096
      kernel.sem = 250 32000 100 128
      net.ipv4.ip_local_port_range = 9000 65500
      net.core.rmem_default = 262144
      net.core.rmem_max = 4194304
      net.core.wmem_default = 262144
      net.core.wmem_max = 1048586

      sysctl -p #使修改生效 -->

- 修改用户资源限制

      echo -e "oracle\tsoft\tnproc\t2047\noracle\thard\tnproc\t16384\noracle\tsoft\tnofile\t1024\noracle\thard\tnofile\t65536\noracle\tsoft\tstack\t10240" >/etc/security/limits.conf
<!-- 
      vi /etc/security/limits.conf

      oracle              soft    nproc  2047
      oracle              hard    nproc  16384
      oracle              soft    nofile  1024
      oracle              hard    nofile  65536
      oracle              soft    stack   10240
 -->
- 创建安装目录

      mkdir -p /usr/local/oracle /usr/local/oraInventory /usr/local/oradata/;\
      chown -R oracle:oinstall /usr/local/oracle /usr/local/oraInventory /usr/local/oradata/;\
      chmod -R 775 /usr/local/oracle /usr/local/oraInventory /usr/local/oradata/

- 创建oraInst.loc

      echo -e "inventory_loc=/usr/local/oraInventory\ninst_group=oinstall" >/etc/oraInst.loc;\
      chown oracle:oinstall /etc/oraInst.loc;\
      chmod 664 /etc/oraInst.loc


<!-- 
      vim .bash_profile
      #添加
      export ORACLE_BASE=/usr/local/oracle
      export ORACLE_HOME=$ORACLE_BASE/product/11.2.0/db_1
      export ORACLE_SID=orcl    
      export ORACLE_OWNER=oracle
      export PATH=$PATH:$ORACLE_HOME/bin:$HOME/bin
      export LD_LIBRARY_PATH=$ORACLE_HOME/lib:/lib:/usr/lib
       -->

## 安装Oracle数据库(oracle用户)

- 配置环境变量

      echo -e "export ORACLE_BASE=/usr/local/oracle\nexport ORACLE_HOME=\$ORACLE_BASE/product/11.2.0/db_1\nexport ORACLE_SID=orcl\nexport ORACLE_OWNER=oracle\nexport PATH=\$PATH:\$ORACLE_HOME/bin:\$HOME/bin\nexport LD_LIBRARY_PATH=\$ORACLE_HOME/lib:/lib:/usr/lib" >>.bash_profile;\
      source .bash_profile
      
- 安装包解压

      # 解压并拷贝rsp文件至指定目录，部分路径需指定
      unzip name.zip
      mv *.rsp /path/to/rsp
      chown oracle:oinstall /path/to/rsp

- 执行安装

      # 路径根据实际情况而定
      /home/database/./runInstaller -silent -force -ignorePrereq -responseFile /usr/local/oracle/db_install.rsp

      dbca -createDatabase -silent -responseFile ./dbca.rsp
      netca /silent /responsefile /path/netca.rsp #斜杠绝对路径（坑）

- 添加用户及表

      # 建立表空间
      CREATE TEMPORARY TABLESPACE "EMCTEMP"
      TEMPFILE  '/usr/local/oradata/emc/emctmp.dbf' SIZE 100 M AUTOEXTEND ON NEXT 100 M MAXSIZE 1 G

      CREATE TABLESPACE "EMCDB"
      DATAFILE  '/usr/local/oradata/emc/emcdb.dbf' SIZE 100 M AUTOEXTEND ON NEXT 100 M MAXSIZE 2 G
      # 创建用户
      CREATE USER "EMC" IDENTIFIED BY "yanghaa" DEFAULT TABLESPACE "EMCDB" TEMPORARY TABLESPACE "EMCTEMP";
      GRANT "CONNECT", "DBA", "RESOURCE" TO "EMC";
      ALTER USER "EMC" QUOTA UNLIMITED ON "EMCDB";
      GRANT CREATE ANY VIEW TO "EMC"
      
      
<!--
      CREATE TABLESPACE "EMCDB"
       DATAFILE  '/usr/local/oradata/emc/EMCDB' SIZE 100 G AUTOEXTEND ON NEXT 100 M MAXSIZE 100 G
      
      CREATE TEMPORARY TABLESPACE "EMCTEMP"
       TEMPFILE  '/usr/local/oradata/emc/EMCTEMP' SIZE 5 G AUTOEXTEND ON NEXT 50 M MAXSIZE 5 G 

      CREATE USER "EMC" IDENTIFIED BY "yanghaa" DEFAULT TABLESPACE "EMCDB" TEMPORARY TABLESPACE "EMCTEMP";

      GRANT "DBA", "RESOURCE" TO "EMC" WITH ADMIN OPTION;

      ALTER USER "EMC" DEFAULT ROLE "DBA";

      ALTER USER "EMC" QUOTA UNLIMITED ON "EMCDB"
      -->

## Buildout (emc用户)

- 准备源代码

      git clone git@github.com:emcupdate/sites.git &&
      cd sites &&
      git clone git@github.com:emcupdate/src.git &&
      git submodule init &&
      git submodule update

- python虚拟环境配置(逐行运行)

      pip install virtualenv --user
      virtual --no-site-packages --no-setuptools ~/venv
      source ~/venv/bin/activate
      python bootstrap-buildout.py --buildout-version=2.5.3 --setuptools-version=26.1.1 -c buildout_dev.cfg
      # 回到sites目录下
      bin/buildout -Nv -c deploy_haproxy.cfg

## 安装静态文件(emc用户)

- 需要注意目录权限

      cd /var/www/html
      sudo git clone git@github.com:emcupdate/emcstatic.git
      # 修改httpd.conf
      sudo vim /etc/httpd/conf/httpd.conf
      # 修改文件root到emcstatic或者直接到httpd目录下
      wget https://raw.githubusercontent.com/emcupdate/emcstatic/master/httpd.conf

- 启动Apache

      sudo systecmctl start httpd

# 安装完成

----

## 其他操作

- supervisord 操作

      bin/supervisorctl shutdown # 关闭supervisord
      bin/supervisorctl taill programname # 显示日志

- test 操作
      
      bin/test -s packagename -m methodname -t testname
      bin/test -s emc.kb -m test_db_tables -t test_create_tables # 运行测试实例，加入test_前缀
      bin/test -s emc.kb -m test_log_db -t test_db_mapping_userlog # 创建日志表
      bin/test -s emc.kb -m test_db_tables --list-tests # 显示所有测试实例

- CentOS 添加EPELRepo and IUSRepo

      sudo rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
      sudo rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-2.el7.elrepo.noarch.rpm
      sudo yum install epel-release             //centos install epelrepo
      sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-$(rpm -E '%{rhel}').noarch.rpm         //REDHAT install EPELrepo

- To install the IUS release package, run the following command:

      sudo yum install https://$(rpm -E '%{?centos:centos}%{!?centos:rhel}%{rhel}').iuscommunity.org/ius-release.rpm

- Upgrade installed packages to IUS versions

      sudo yum install yum-plugin-replace

- The plug-in provides a yum replace command that replaces a specified package and installs any required dependencies at the same time. For example, to replace the installed PHP package with the PHP 5.6 package from the IUS repository, run the following command:

      sudo yum replace php --replace-with php56u
----
      


      sudo yum install binutils-devel* compat-libcap1.* gcc gcc-c++ glibc-static.* glibc-utils.x86_64 ksh libaio libaio-devel libgcc libstdc++-* libXi libXtst* make
### 图形化安装oracle数据库

解决汉语安装器乱码

      export NLS_LANG=american_america.ZHS16GBK
      export LC_ALL=C
      ./runInstaller

#### pip

pippackage

      pip install --user virtualenv
      pip install --no-index -f ~/pippackage ***
buildoutcache

      [buildout]
      eggs-directory = ~/buildout/eggs
      download-cache = ~/buildout/download-cache
      abi-tag-eggs = true

instantclient-basic-linux

      sudo sh -c "echo /path/instantclient_11_2 > /etc/ld.so.conf.d/oracle-instantclient.conf"
      sudo ldconfig  # 或者设置环境变量 LD_LIBRARY_PATH

### Apache

修复403错误

      restorecon -v -R /var/www/html/
      