FROM amazonlinux:latest

ARG USER

RUN yum -y update && yum -y install openssh-server && yum -y install sudo
RUN adduser $USER && mkdir -p /home/$USER/.ssh && chown $USER:$USER /home/$USER/.ssh && chmod 0700 /home/$USER/.ssh
ADD ./_authorized_keys/id_rsa.pub /home/$USER/.ssh/authorized_keys
RUN chown $USER:$USER /home/$USER/.ssh/authorized_keys && chmod 0600 /home/$USER/.ssh/authorized_keys
RUN echo "$USER ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers

CMD /etc/init.d/sshd start && tail -f /dev/null

# Import the Centos-7 RPM GPG key to prevent warnings
#RUN rpm --import http://mirror.centos.org/centos/RPM-GPG-KEY-CentOS-7
#RUN rpm --import http://dl.fedoraproject.org/pub/epel/RPM-GPG-KEY-EPEL-7

# system update
RUN yum -y update && yum clean all

# set locale
RUN yum reinstall -y glibc-common && yum clean all
RUN localedef -f UTF-8 -i ja_JP ja_JP.UTF-8
ENV LANG ja_JP.UTF-8
ENV LANGUAGE ja_JP:ja
ENV LC_ALL ja_JP.UTF-8
RUN unlink /etc/localtime
RUN ln -s /usr/share/zoneinfo/Japan /etc/localtime

# ===============================================================================
# BASE packages
# ===============================================================================
RUN yum --enablerepo=extras clean metadata
RUN yum install -y zlib zlib-devel make gcc gcc-c++ openssl openssl-devel readline-devel pcre pcre-devel
RUN yum install -y openssh openssh-server
RUN yum install -y net-tools wget sudo
RUN yum install -y tar zip unzip bzip2 which tree
RUN yum install -y git

# ===============================================================================
# Install Applications
# ===============================================================================

# epel,remi
RUN yum install -y epel-release && \
	yum install -y http://rpms.famillecollet.com/enterprise/remi-release-7.rpm && \
    yum clean all && \
	sed -i -e "s/enabled *= *1/enabled=0/g" /etc/yum.repos.d/epel.repo && \
	sed -i -e "s/enabled *= *1/enabled=0/g" /etc/yum.repos.d/remi.repo

# httpd, sshd, scp, openssl, sudo, which, mod_ssl
RUN yum install -y httpd httpd-tools openssh-server openssh-clients openssl sudo which mod_ssl && \
    yum clean all

# libmcrypt, supervisor
RUN yum install --enablerepo=epel -y libmcrypt supervisor && \
    yum clean all

# gd-last (for php-gd)
RUN yum install --enablerepo=remi -y gd-last && \
    yum clean all

# php
RUN yum -y install --enablerepo=epel,remi,remi-php72 php php-devel php-gd php-mcrypt php-mysqlnd php-opcache php-mysql php-pdo php-phar php-mbstring php-zlib php-ctype php-session php-json php-xml php-openssl php-zip php-curl php-fileinfo php-intl php-hash php-zend && \
    yum clean all && \
	sed -i -e "s/;date.timezone *=.*$/date.timezone = Asia\/Tokyo/" /etc/php.ini

# php-pecl-memcached
RUN yum install --enablerepo=remi,remi-php72 -y php-pecl-memcached && \
    yum clean all

# composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

# phpunit
RUN curl -L https://phar.phpunit.de/phpunit.phar > /usr/local/bin/phpunit && \
	chmod +x /usr/local/bin/phpunit

# initialize for ssh
#RUN sed -i '/pam_loginuid\.so/s/required/optional/' /etc/pam.d/sshd && \
#	ssh-keygen -A

# postfix
#RUN yum -y install postfix

# SES
#RUN yum -y install cyrus-sasl-plain
#RUN yum -y install cyrus-sasl-md5

# let's encrypt
#RUN cd /etc
#RUN git clone https://github.com/letsencrypt/letsencrypt
#RUN cd /etc/letsencrypt
#RUN ./letsencrypt-auto --help

#CMD /bin/bash -c "cd /var/www/e-earphone-eccube4 \ composer update"

# crontab
RUN yum -y install cronie

# rsyslog
RUN yum -y install rsyslog
RUN echo '[ -f /usr/sbin/rsyslogd ] && /usr/sbin/rsyslogd' >> $HOME/.bash_profile

RUN sed -i -e '/$ModLoad imjournal/s/^/#/' /etc/rsyslog.conf
RUN sed -i -e 's/$OmitLocalLogging on/$OmitLocalLogging off/' /etc/rsyslog.conf
RUN sed -i -e '/$IMJournalStateFile/s/^/#/' /etc/rsyslog.conf
RUN sed -i -e '/$SystemLogSocketName/s/^/#/' /etc/rsyslog.d/listen.conf

# enable
RUN systemctl enable httpd
RUN systemctl enable crond
# RUN systemctl enable postfix
RUN systemctl enable rsyslog

CMD /bin/bash -c "httpd && php-fpm && tail -f /dev/null"
