## 安装说明

### emcsite and oracle系统级依赖包
- dnf

      dnf install git gcc-c++ patch openssl-devel libjpeg-devel  libxslt-devel make which python-devel  readline-devel wv poppler-utils binutils gcc gcc-gfortran make m4 perl tar git perl-ExtUtils-MakeMaker texlive httpd compat-libstdc++-33 compat-libstdc++-33.i686 elfutils-libelf elfutils-libelf-devel glibc glibc.i686 glibc-common glibc-devel glibc-devel.i686 glibc-headers ksh libaio libaio.i686 libaio-devel libaio-devel.i686 libgcc libgcc.i686 libstdc++ libstdc++.i686 libstdc++-devel libXi libXtst sysstat unixODBC unixODBC-devel openssl python-docutils

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
      