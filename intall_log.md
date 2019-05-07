## 安装说明

### emcsite and oracle系统级依赖包
- dnf

      dnf install git gcc-c++ patch openssl-devel libjpeg-devel  libxslt-devel make which python-devel  readline-devel wv poppler-utils binutils gcc gcc-gfortran make m4 perl tar git perl-ExtUtils-MakeMaker texlive httpd compat-libstdc++-33 compat-libstdc++-33.i686 elfutils-libelf elfutils-libelf-devel glibc glibc.i686 glibc-common glibc-devel glibc-devel.i686 glibc-headers ksh libaio libaio.i686 libaio-devel libaio-devel.i686 libgcc libgcc.i686 libstdc++ libstdc++.i686 libstdc++-devel libXi libXtst sysstat unixODBC unixODBC-devel openssl python-docutils python-pip

      #dnf install openssl openssl-devel

### 创建用户与组

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

      sysctl -p #使修改生效

- 修改用户资源限制

      vi /etc/security/limits.conf

      oracle              soft    nproc  2047
      oracle              hard    nproc  16384
      oracle              soft    nofile  1024
      oracle              hard    nofile  65536
      oracle              soft    stack   10240

- 创建安装目录

      mkdir -p /usr/local/oracle /usr/local/oraInventory /usr/local/oradata/;\
      chown -R oracle:oinstall /usr/local/oracle /usr/local/oraInventory /usr/local/oradata/;\
      chmod -R 775 /usr/local/oracle /usr/local/oraInventory /usr/local/oradata/

- 创建oraInst.loc

      vi /etc/oraInst.loc
      #文件内加入以下内容
      echo -e "inventory_loc=/usr/local/oraInventory\ninst_group=oinstall" >/etc/oraInst.loc
      chown oracle:oinstall /etc/oraInst.loc
      chmod 664 /etc/oraInst.loc

- 配置环境变量

      vim .bash_profile
      #添加
      export ORACLE_BASE=/usr/local/oracle
      export ORACLE_HOME=$ORACLE_BASE/product/11.2.0/db_1
      export ORACLE_SID=orcl    
      export ORACLE_OWNER=oracle
      export PATH=$PATH:$ORACLE_HOME/bin:$HOME/bin
      export LD_LIBRARY_PATH=$ORACLE_HOME/lib:/lib:/usr/lib
      
      echo -e "export ORACLE_BASE=/usr/local/oracle\nexport ORACLE_HOME=\$ORACLE_BASE/product/11.2.0/db_1\nexport ORACLE_SID=orcl\nexport ORACLE_OWNER=oracle\nexport PATH=\$PATH:\$ORACLE_HOME/bin:\$HOME/bin\nexport LD_LIBRARY_PATH=\$ORACLE_HOME/lib:/lib:/usr/lib" >>.bash_profile
      #运行source生效
      source .bash_profile

- 执行安装

      /home/database/./runInstaller -silent -force -ignorePrereq -responseFile /usr/local/oracle/db_install.rsp

      dbca -createDatabase -silent -responseFile ./dbca.rsp
      netca /silent /responsefile /path/netca.rsp #斜杠绝对路径（坑）

- 添加用户及表

      # 建立表空间
      CREATE TABLESPACE "EMCDB"
       DATAFILE  '/usr/local/oradata/emc/EMCDB' SIZE 100 G AUTOEXTEND ON NEXT 100 M MAXSIZE 100 G
      
      CREATE TEMPORARY TABLESPACE "EMCTEMP"
       TEMPFILE  '/usr/local/oradata/emc/EMCTEMP' SIZE 5 G AUTOEXTEND ON NEXT 50 M MAXSIZE 5 G 

      CREATE USER "EMC" IDENTIFIED BY "yanghaa" DEFAULT TABLESPACE "EMCDB" TEMPORARY TABLESPACE "EMCTEMP";

      GRANT "DBA", "RESOURCE" TO "EMC" WITH ADMIN OPTION;

      ALTER USER "EMC" DEFAULT ROLE "DBA";

      ALTER USER "EMC" QUOTA UNLIMITED ON "EMCDB"


### buildout 

- 准备源代码

- python虚拟环境配置

      pip install virtualenv --user
      virtual --no-site-packages --no-setuptools venv
      source venv/bin/activate
      



### CentOS 添加EPELRepo and IUSRepo

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
      