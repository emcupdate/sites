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
      