NextCloud 개념잡기
====

1.NextCloud 란?
----
1. NextCloud는 파일 호스팅 서비스를 만들고 사용하기 위한 client-server software이며, 오픈소스로 알려진 인프라 구축을 진행함.

2. 개인 서버 장치에 설치하여 운영할 수 있고 Office , Google Drive , MatterMost , Calendar 통합된 기능이면서
로컬 컴퓨터 또는 외부 파일 스토리지 호스팅에 사용할 수 있어 NAS 기술을 접하는 사람으로써 Storage Cloud 서비스에 적합
하다고 판단하여 진행함.

3. 원래는 OwnCloud 개발자인 Frank Karlitschek은 OwnCloud에서 파생된 NextCloud를 만들었다.

4. 웹이나 앱에서 서버의 파일을 사용하고, 파일을 전송하고, 파일을 외부와 공유할 Url을 만든다.

5. 모바일 기기에서 촬영한 사진을 자동으로 업로드, WebDav 프로토콜과 채팅의 기능도 제공한다.

![Nextcloud](/image/1.JPG)

2.NAS와Cloud의 조합
----

**NAS(Network Attach Storage)를 많은 사용자의 관점에서 단순히 백업용으로 생각하고 사용하고 있다. NAS는 파일 관리에 적합하여 
기능에 대표적인 하나로 클라우드 서비스도 제공을 한다.**
**Petabyte나 Exabyte 같은 단위가 이제는 기업에서 폭발적으로 사용하는 데이터 활용에 있어서 대응 및 서비스를 하기 위해
활용성이 떨어지는 데이터와 활용성이 높은 데이터를 분류할 수도 있습니다.**
**NAS는 '네트워크 드라이브'나 '외부 저장장치의' 형태로만 사용되고 Cloud 서비스를 이용하면 '동기화 방식'이 기본으로 되어
용량이 늘어나면서 NAS와 같이 활용되는 목적으로 구축된다고 합니다.**
**NAS는 하드디스크에 의존하고, Raid구성으로 비용도 많이 드는데, 클라우드 서비스는 휴지통이나 , 파일 히스토리 기능을 지원하여
복구가 수월하다고 생각되어 안정성이 보장된다고 생각이 된다.**

![NextCloud](/image/2.JPG)

3.NextCloud 설치과정
----
* CentOS 7.9 , PHP7.2 , MariaDB 배포하고 Apache에서 실행되는 Nextcloud를 배포한다.
* CentOS 7 설치하여 성공적인 플랫폼을 제공하도록 구축 하였다.
<pre>
<code>
~]# yum update -y * 최신 커널 업데이트를 진행한다.
~]# yum install -y httpd * Apache 웹 서버 패키지를 설치한다.
</code>
</pre>

#### 3.1 Apache 설정 -> vim /etc/httpd/conf.d/nextcloud.conf
<pre>
<code>
<> -> {}로 표기
{VirtualHost *:80} 
  DocumentRoot /var/www/html  // 실제 소스파일이 있는 Web Root 지정소
  ServerName test.domain.com  // 서비스 할 도메인 지정소
{Directory "/var/www/html/"}
  Require all granted  // 디렉터리에 대한 액세스 허용
  AllowOverride All    // 접근자를 모두 허락하도록 설정
  Options FollowSymLinks MultiViews  // 디렉터리에 대한 특정 옵션 설정
{/Directory}
{/VirtualHost}
</code>
</pre>
~]# systemctl enable httpd.service

~]# systemctl start httpd.service
* Apache Web Service 데몬 enable과 start 시킨다.

#### 3.2 PHP 관련 패키지 설치
* NextCloud는 PHP 기반 프로그래밍 언어를 사용하여 새로운 모듈을 만들고 싶으면 (최신)7.2이상의 패키지를 설치한다.
<pre>
<code>
~]# yum install -y centos-release-scl
rh-php72 rh-php72-php rh-php72-php-gd rh-php72-php-mbstring \
rh-php72-php-intl rh-php72-php-pecl-apcu rh-php72-php-mysqlnd rh-php72-php-pecl-redis \
rh-php72-php-opcache rh-php72-php-imagick
</code>
</pre>

#### 3.3 Apache 설정 파일 SymLinks 
<pre>
<code>
~]# ln –s /opt/rh/httpd24/root/etc/httpd/conf.d/rh-php72-php.conf /etc/httpd/conf.d/
~]# ln -s /opt/rh/httpd24/root/etc/httpd/conf.modules.d/15-rh-php72-php.conf /etc/httpd/conf.modules.d/
~]# ln -s /opt/rh/httpd24/root/etc/httpd/modules/librh-php72-php7.so /etc/httpd/modules/
</code>
</pre>
* httpd 설치 경로 /opt/rh/httpd24에서 Nextcloud 설정 경로와 심볼릭 링크를 구성 하였다.

#### 3.4 MariaDB 설치
<pre>
<code>
~]# yum install -y mariadb mariadb-server
~]# systemctl enable mariadb.service
~]# systemctl start mariadb.service
</code>
</pre>
* 이 작업을 완료 한 후 Nextcloud가 액세스 할 수 있도록 사용자 이름과 비밀번호로 데이터 베이스를 생성합니다.

#### 3.5 Nextcloud 설치
1. [NextCloud Install](https://nextcloud.com/install/)
* NextCloud Server 다운로드 -> 다운로드 -> 서버 소유자 용 아카이브 파일로 이동하여 tar.bz2 다운로드 하였다. 
2. GPG-PUB-KEY Error로 인한 yum install 실패 조치 사항
<pre>
<code>
~]# wget https://download.nextcloud.com/server/release/nextcloud-20.0.4.tar.bz2.asc
~]# wget https://nextcloud.com/nextcloud.asc
~]# gpg -import nextcloud.asc
~]# gpg –verify nextcloud-20.0.4.tar.bz2.asc nextcloud-20.0.4.tar.bz2
</code>
</pre>
3. RPM 기반의 패키지들은 RPM-GPG-KEY라는 공개키 기반의 디지털 서명과 검증을 통해 해당 패키지의 버전과 그에 따른 
보증과 검증을 수행하여 때문에 Public GPG-KEY가 등록 되어있지 않은 상태에서는 yum을 사용할 수 없기 때문에 등록한다.

#### 3.6 압축 풀기
<pre>
<code>
~]# bunzip2 nextcloud-*.bz2
~]# tar -xvf nextcloud-20.0.4.tar
~]# cp -R nextcloud /var/www/html/
</code>
</pre>
* 콘텐츠를 웹 서버의 루트 디렉터리로 복사한다. 아파치 사용으로 /var/www/html 입니다.

<pre>
<code>
~]# mkdir /var/www/html/nextcloud/data
</code>
</pre>
* 설치 프로세스 중에는 데이터 폴더가 생성되지 않아 수동으로 디렉터리를 생성하였다.

<pre>
<code>
~]# Chown -R apache:apache /var/www/html/nextcloud
~]# systemctl restart httpd.service
</code>
</pre>
* nextcloud 전체 폴데에 Apache 사용자, 그룹 권한을 부여하고 데몬을 재시작한다.

<pre>
<code>
~]# firewall-cmd --zone=public --add-service=http --permanent
~]# firewall-cmd --reload
</code>
</pre>
* Apache 를 접근하도록 방화벽에 설정하고 활성화 시킨다.

#### 3.6 웹 접속 URL 확인
1. Nextcloud 설치가 끝났으면 /var/www/html/nextcloud/config/config.php 파일에서 웹 접근을 위한 경로를 확인한다.
~]# vim /var/www/html/nextcloud/config/config.php
<pre>
<code>
<?php
$CONFIG = array (
  'instanceid' => 'oc5qa6uhr5qj',
  'passwordsalt' => 'lSEQRm75mSF9MpG8y4zPBHrF+D+A1B',
  'secret' => 'Id0gGxWofZbINYLGouTjLeJJh36eOOsBY/lch1w532Sx5/m0',
  'trusted_domains' =>
  array (
    0 => '192.168.1.15',
  ),
  'datadirectory' => '/var/www/html/nextcloud/data',
  'dbtype' => 'sqlite3',
  'version' => '20.0.4.0',
  'overwrite.cli.url' => 'http://192.168.xx.xx/nextcloud', // URL 주소를 확인할 수 있다.
  'installed' => true,
  'memcache.distributed' => '\\OC\\Memcache\\Redis',
  'memcache.locking' => '\\OC\\Memcache\\Redis',
  'memcache.local' => '\\OC\\Memcache\\APCu',
  'redis' =>
  array (
    'host' => 'localhost',
    'port' => 6379,
  ),
  'maintenance' => false,
);
</code>
</pre>
* URL 을 통해 접근하면 관리자 웹 화면이 나타난다.

![Nextcloud](/image/3.png)
