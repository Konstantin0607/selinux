# ��������� Vagrantfile

           vagrant up


#1.��������� nginx �� ������������� ����� 3-�� ������� ���������:
#��� ������ ��������� 

           yum install -y nginx

#��� ���� ����� �������� � ���������� ��������� semanage:

           yum -y install policycoreutils-python

#1.1 ������������� setsebool;
# � ���������������� ����� nginx ������ ����������� ���� �� 12345

           nano /etc/nginx/nginx.conf

#��������� nginx, ������� ������ � ����� ������:

            systemctl status nginx.service

#�����
           Mar 12 06:09:44 SELinux systemd[1]: Starting The nginx HTTP and reverse proxy server...
           Mar 12 06:09:44 SELinux nginx[3960]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
           Mar 12 06:09:44 SELinux nginx[3960]: nginx: [emerg] bind() to 0.0.0.0:12345 failed (13: Permission denied)
           Mar 12 06:09:44 SELinux nginx[3960]: nginx: configuration file /etc/nginx/nginx.conf test failed

#����� ��������� �������, ��� ���� ����� ������ ����� ���������� ��� ��������

           audit2why < /var/log/audit/audit.log


#������� audit2why ����������� ��������� ������� setsebool -P nis_enabled 1
#���� -P �������� ������� � ����� ������������. ��������� ������� ��� ����� -P

           setsebool nis_enabled 1

#����� ����� ���-������ nginx ������� �����������,��������� ��� � ������� ������

           systemctl start nginx.service

           systemctl status nginx.service


#�����

            nginx.service - The nginx HTTP and reverse proxy server
            Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; vendor preset: disabled)
            Active: active (running) since Fri 2021-03-12 06:26:05 UTC; 2s ago
            Process: 25158 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
            Process: 25156 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
            Process: 25154 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
            Main PID: 25160 (nginx)
            CGroup: /system.slice/nginx.service
                  +-25160 nginx: master process /usr/sbin/nginx
                  L-25161 nginx: worker process


#1.2 ���������� �������������� ����� � ��������� ���;
#������� ����� ����� ����� �������� �� http, ����� ��� ������ ����� ��� ���

           semanage port -l | grep http

#�����

           http_cache_port_t              tcp      8080, 8118, 8123, 10001-10010
           http_cache_port_t              udp      3130
           http_port_t                    tcp      80, 81, 443, 488, 8008, 8009, 8443, 9000
           pegasus_http_port_t            tcp      5988
           pegasus_https_port_t           tcp      5989

#��������� ��� ������������� ���� � ������� ��������

           semanage port -a -t http_port_t -p tcp 12345

#��������� ����� ��������� ��� http

           semanage port -l | grep http

#�����

           http_cache_port_t              tcp      8080, 8118, 8123, 10001-10010
           http_cache_port_t              udp      3130
           http_port_t                    tcp      12345, 80, 81, 443, 488, 8008, 8009, 8443, 9000
           pegasus_http_port_t            tcp      5988
           pegasus_https_port_t           tcp      5989


#������ ���-������ nginx ����������� �� ����� ������������� �����, ���������

           systemctl start nginx.service

           systemctl status nginx.service

#�����

           ? nginx.service - The nginx HTTP and reverse proxy server
             Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; vendor preset: disabled)
            Active: active (running) since Fri 2021-03-12 06:45:55 UTC; 9s ago
             Process: 1164 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
             Process: 1162 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
             Process: 1160 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
           Main PID: 1166 (nginx)
           CGroup: /system.slice/nginx.service
                 +-1166 nginx: master process /usr/sbin/nginx
                 L-1167 nginx: worker process

#1.3 ������������ � ��������� ������ SELinux
# ����������� ������ �� ������ ����� ������

#��������� �������

           audit2allow -M httpd_add --debug < /var/log/audit/audit.log


#�����
    
            audit2allow -M httpd_add --debug < /var/log/audit/audit.log
            ******************** IMPORTANT ***********************
            To make this policy package active, execute:

            semodule -i httpd_add.pp

             ls -l
             total 24
             -rw-------. 1 root root 5570 Apr 30  2020 anaconda-ks.cfg
             -rw-r--r--. 1 root root  964 Mar 12 06:51 httpd_add.pp
             -rw-r--r--. 1 root root  247 Mar 12 06:51 httpd_add.te
             -rw-------. 1 root root 5300 Apr 30  2020 original-ks.cfg


#����������� ��� ��������� ������ � ���������

           semodule -i httpd_add.pp

           semodule -l | grep http

#����� 
         
           httpd_add       1.0

#��� ���-������ ������ ������  �������� �� ����� 12345,���������


           systemctl start nginx.service

           systemctl status nginx.service

#�����
     
            nginx.service - The nginx HTTP and reverse proxy server
            Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; vendor preset: disabled)
           Active: active (running) since Fri 2021-03-12 07:12:01 UTC; 5s ago
            Process: 986 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
            Process: 984 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
            Process: 982 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
           Main PID: 988 (nginx)
           CGroup: /system.slice/nginx.service
                 +-988 nginx: master process /usr/sbin/nginx
                 L-989 nginx: worker process


#2.���������� ����������������� ���������� ��� ���������� selinux
#������������ � ���������� ������ vagrant ssh client � ������� ��������� �������:

           nsupdate -k /etc/named.zonetransfer.key
           server 192.168.50.10
           zone ddns.lab 
           update add www.ddns.lab. 60 A 192.168.50.15
           send

# �������� ������ ���������� �� �������: SERVFAIL 
# ��� ����, ����� ������ ��� ������, ���������� ������� ��������� ���������� �������
# �� ���������� ������� (��������� ������ ���� ������ SELINUX ���������� ������) �� ������� DNS,
# ������� ����������� � ������ / var / log / audit / audit .log � / var / log / messages,
# � ����� ��� ������������ ������ � ����������� systemctl status � ������,
# ����� ����������� �������� BIND �������, ��� ����� ������������ ������.

# ����� �������� ����� ��������: 

#1.��� ����� ������ ��� ���������� � ������ �� ����� SELINUX,
# ����� ������� ������������ ��������� ��������

           audit2why < /var/log/audit/audit.log

#�����

           root@ns01 vagrant]# audit2why < /var/log/audit/audit.log
           type=AVC msg=audit(1587231618.482:1955): avc:  denied  { search } for  pid=7268 comm="isc-worker0000" name="net" dev="proc" ino=33134 scontext=system_u:system_r:named_t:s0 tcontext=system_u:object_r:sysctl_net_t:s0 tclass=dir permissive=0

                   Was caused by:
	                   Missing type enforcement (TE) allow rule.

	                   You can use audit2allow to generate a loadable module to allow this access.

           type=AVC msg=audit(1587231618.482:1956): avc:  denied  { search } for  pid=7268 comm="isc-worker0000" name="net" dev="proc" ino=33134 scontext=system_u:system_r:named_t:s0 tcontext=system_u:object_r:sysctl_net_t:s0 tclass=dir permissive=0

                   Was caused by:
	                   Missing type enforcement (TE) allow rule.

	                   You can use audit2allow to generate a loadable module to allow this access. 


#����� ��������� �������:

           audit2allow -M named-selinux --debug < /var/log/audit/audit.log 

           semodule -i named-selinux.pp


# �������� �� ����, ����� ����� ������:

            [root@ns01 vagrant]# cat /var/log/messages | grep ausearch
            Mar 12 07:27:01 localhost python: SELinux is preventing /usr/sbin/named from search access on the directory net.#012#012*****  Plugin catchall (100. confidence) suggests   **************************#012#012If you believe that named should be allowed search access on the net directory by default.#012Then you should report this as a bug.#012You can generate a local policy module to allow this access.#012Do#012allow this access for now by executing:#012# ausearch -c 'isc-worker0000' --raw | audit2allow -M my-iscworker0000#012# semodule -i my-iscworker0000.pp#012

               
# ��������� �������:

           ausearch -c 'isc-worker0000' --raw | audit2allow -M my-iscworker0000 | semodule -i my-iscworker0000.pp


# �������� ��� � �� ��������, ������� ������ ��� ����� ��� / var / log / messages:
 
           Mar 12 07:29:47 localhost python: SELinux is preventing /usr/sbin/named from read access on the file ip_local_port_range.#012#012*****  Plugin catchall (100. confidence) suggests   **************************#012#012If you believe that named should be allowed read access on the ip_local_port_range file by default.#012Then you should report this as a bug.#012You can generate a local policy module to allow this access.#012Do#012allow this access for now by executing:#012# ausearch -c 'isc-worker0000' --raw | audit2allow -M my-iscworker0000#012# semodule -i my-iscworker0000.pp#012
           

# ����� � ����� DNS ������� ��� ������� ��������� ���� ip_local_port_range 
# ��������� �������:

           ausearch -c 'isc-worker0000' --raw | audit2allow -M my-iscworker0001 | semodule -i my-iscworker0001.pp

# �� �������� ���� ������:

           Mar 12 07:34:27 localhost python: SELinux is preventing /usr/sbin/named from open access on the file /proc/sys/net/ipv4/ip_local_port_range.#012#012*****  Plugin catchall (100. confidence) suggests   **************************#012#012If you believe that named should be allowed open access on the ip_local_port_range file by default.#012Then you should report this as a bug.#012You can generate a local policy module to allow this access.#012Do#012allow this access for now by executing:#012# ausearch -c 'isc-worker0000' --raw | audit2allow -M my-iscworker0000#012# semodule -i my-iscworker0000.pp#012

# ����� ��������� �������:

           ausearch -c 'isc-worker0000' --raw | audit2allow -M my-iscworker0002 | semodule -i my-iscworker0002.pp

# �� �������� ���� ������:

           Mar 12 07:39:18 localhost python: SELinux is preventing /usr/sbin/named from getattr access on the file /proc/sys/net/ipv4/ip_local_port_range.#012#012*****  Plugin catchall (100. confidence) suggests   **************************#012#012If you believe that named should be allowed getattr access on the ip_local_port_range file by default.#012Then you should report this as a bug.#012You can generate a local policy module to allow this access.#012Do#012allow this access for now by executing:#012# ausearch -c 'isc-worker0000' --raw | audit2allow -M my-iscworker0000#012# semodule -i my-iscworker0000.pp#012


# ����� ��������� �������:

           ausearch -c 'isc-worker0000' --raw | audit2allow -M my-iscworker0003 | semodule -i my-iscworker0003.pp


# �� ��������, ���� ������:

           Mar 12 07:44:51 localhost python: SELinux is preventing isc-worker0000 from create access on the file named.ddns.lab.view1.jnl.#012#012*****  Plugin catchall_labels (83.8 confidence) suggests   *******************#012#012If you want to allow isc-worker0000 to have create access on the named.ddns.lab.view1.jnl file#012Then you need to change the label on named.ddns.lab.view1.jnl#012Do#012# semanage fcontext -a -t FILE_TYPE 'named.ddns.lab.view1.jnl'#012where FILE_TYPE is one of the following: dnssec_trigger_var_run_t, ipa_var_lib_t, krb5_host_rcache_t, krb5_keytab_t, named_cache_t, named_log_t, named_tmp_t, named_var_run_t, named_zone_t.#012Then execute:#012restorecon -v 'named.ddns.lab.view1.jnl'#012#012#012*****  Plugin catchall (17.1 confidence) suggests   **************************#012#012If you believe that isc-worker0000 should be allowed create access on the named.ddns.lab.view1.jnl file by default.#012Then you should report this as a bug.#012You can generate a local policy module to allow this access.#012Do#012allow this access for now by executing:#012# ausearch -c 'isc-worker0000' --raw | audit2allow -M my-iscworker0000#012# semodule -i my-iscworker0000.pp#012


# ����� ��������� �������:

           ausearch -c 'isc-worker0000' --raw | audit2allow -M my-iscworker0004 | semodule -i my-iscworker0004.pp


# ����� ���� � ����� / var / log / messages ��������� ����������, �� ������ ���� � ������ ������� systemctl status � ������:

           [root@ns01 vagrant]# systemctl status named
           ? named.service - Berkeley Internet Name Domain (DNS)
           Loaded: loaded (/usr/lib/systemd/system/named.service; enabled; vendor preset: disabled)
           Active: active (running) since Sat 2021-03-12 07:51:32 UTC; 4s ago
           Process: 7015 ExecStop=/bin/sh -c /usr/sbin/rndc stop > /dev/null 2>&1 || /bin/kill -TERM $MAINPID (code=exited, status=0/SUCCESS)
           Process: 7028 ExecStart=/usr/sbin/named -u named -c ${NAMEDCONF} $OPTIONS (code=exited, status=0/SUCCESS)
           Process: 7026 ExecStartPre=/bin/bash -c if [ ! "$DISABLE_ZONE_CHECKING" == "yes" ]; then /usr/sbin/named-checkconf -z "$NAMEDCONF"; else echo "Checking of zone files is disabled"; fi (code=exited, status=0/SUCCESS)
           Main PID: 7031 (named)
           CGroup: /system.slice/named.service
	               	   L-7031 /usr/sbin/named -u named -c /etc/named.conf

           Mar 12 07:51:32 ns01 named[7031]: automatic empty zone: view default: HOME.ARPA
           Mar 12 07:51:32 ns01 named[7031]: none:104: 'max-cache-size 90%' - setting to 211MB (out of 235MB)
           Mar 12 07:51:32 ns01 named[7031]: command channel listening on 192.168.50.10#953
           Mar 12 07:51:32 ns01 named[7031]: managed-keys-zone/view1: journal file is out of date: removing journal file
           Mar 12 07:51:32 ns01 named[7031]: managed-keys-zone/view1: loaded serial 10
           Mar 12 07:51:32 ns01 named[7031]: managed-keys-zone/default: journal file is out of date: removing journal file
           Mar 12 07:51:32 ns01 named[7031]: managed-keys-zone/default: loaded serial 10
           Mar 12 07:51:32 ns01 named[7031]: zone 0.in-addr.arpa/IN/view1: loaded serial 0
           Mar 12 07:51:32 ns01 named[7031]: zone ddns.lab/IN/view1: journal rollforward failed: no more
           Mar 12 07:51:32 ns01 named[7031]: zone ddns.lab/IN/view1: not loaded due to errors.

# ������� ���� /etc/ named/dynamic/ named.ddns.lab.view1.jnl,
# ������������� ������ DNS ������� systemctl restart named � ������ �� ����� ������, 
# ������ ������������ ���������� ����������� �������.

           [root@ns01 vagrant]# systemctl status named
           ? named.service - Berkeley Internet Name Domain (DNS)
           Loaded: loaded (/usr/lib/systemd/system/named.service; enabled; vendor preset: disabled)
           Active: active (running) since Sat 2021-03-12 07:54:12 UTC; 3min 38s ago
           Process: 7057 ExecStop=/bin/sh -c /usr/sbin/rndc stop > /dev/null 2>&1 || /bin/kill -TERM $MAINPID (code=exited, status=0/SUCCESS)
           Process: 7070 ExecStart=/usr/sbin/named -u named -c ${NAMEDCONF} $OPTIONS (code=exited, status=0/SUCCESS)
           Process: 7068 ExecStartPre=/bin/bash -c if [ ! "$DISABLE_ZONE_CHECKING" == "yes" ]; then /usr/sbin/named-checkconf -z "$NAMEDCONF"; else echo "Checking of zone files is disabled"; fi (code=exited, status=0/SUCCESS)
           Main PID: 7072 (named)
           CGroup: /system.slice/named.service
		           L-7072 /usr/sbin/named -u named -c /etc/named.conf

           Mar 12 07:54:12 ns01 named[7072]: automatic empty zone: view default: 8.B.D.0.1.0.0.2.IP6.ARPA
           Mar 12 07:54:12 ns01 named[7072]: automatic empty zone: view default: EMPTY.AS112.ARPA
           Mar 12 07:54:12 ns01 named[7072]: automatic empty zone: view default: HOME.ARPA
           Mar 12 07:54:12 ns01 named[7072]: none:104: 'max-cache-size 90%' - setting to 211MB (out of 235MB)
           Mar 12 07:54:12 ns01 named[7072]: command channel listening on 192.168.50.10#953
           Mar 12 07:54:12 ns01 named[7072]: managed-keys-zone/view1: journal file is out of date: removing journal file
           Mar 12 07:54:12 ns01 named[7072]: managed-keys-zone/view1: loaded serial 11
           Mar 12 07:54:58 ns01 named[7072]: client @0x7f61fc09df00 192.168.50.15#26071/key zonetransfer.key: view view1: signer "zonetransfer.key" approved
           Mar 12 07:54:58 ns01 named[7072]: client @0x7f61fc09df00 192.168.50.15#26071/key zonetransfer.key: view view1: updating zone 'ddns.lab/IN': adding an RR at 'www.ddns.lab' A 192.168.50.15
           Mar 12 07:55:02 ns01 named[7072]: client @0x7f61fc09df00 192.168.50.15#26071/key zonetransfer.key: view view1: signer "zonetransfer.key" approved


# ����� �� ������ ����� �������:
������� ������������������� ��������� ���������� ����������� � ��� ��� Selinux ���������� ������ � ���������� ������ ������������� ���������� ��� DNS �������, 
� ����� � ��������� ������ ��, � ������� DNS ������ (/usr/sbin/named) ���������� �� ����� ����� ������ (������� ����, ����� �� ����� �������). 
����� ���� ������������� ������� ���� � ����������� .jnl (��� ��������� �������� ������������), ���� ������������ ������������ ���������� ����. 
��� ��� ������ ��� ������ �������� � .jnl ����, ��� ������� ������������ �� ��������� ���� tmp, ��� �������� ����� ����������� ���������� (��� � ���� ������), ������� tmp ����� ����� ������������� ��� ������� ��� ��������� �� �������� ������������. 
��������� ����� ����� �����:

[root@ns01 vagrant]# ls -l /etc/named/dynamic/
total 32
-rw-rw-rw-. 1 named named 509 Mar 12 07:31 named.ddns.lab
-rw-rw-rw-. 1 named named 509 Mar 12 07:31 named.ddns.lab.view1
-rw-r--r--. 1 named named 700 Mar 12 08:21 named.ddns.lab.view1.jnl
-rw-r--r--. 1 named named 348 Mar 12 07:30 tmp-6OGP6YASy1
-rw-r--r--. 1 named named 348 Mar 12 08:34 tmp-HUsH1RRHBF
-rw-r--r--. 1 named named 348 Mar 12 08:07 tmp-OEmMkfw6J6
-rw-r--r--. 1 named named 348 Mar 12 08:59 tmp-R8cPmFCasl
-rw-r--r--. 1 named named 348 Mar 12 08:57 tmp-csgM4QDJR7


#������ ��� ����� ������������ ��� ���������� ������� SELINUX ��� ��������� ��������� ������������ ��� ������ SELINUX
#(�������� ��������� �� ������ � �����������ls -lZ),
#� ������� ���������� BIND, ��� ��� ������� ���������� ������������� �������������� Selinux ��� ���� ����� ������ �������� ��������.
       
#����� ������ �������� ���������� ����� �������:

#��������� Selinux ������ (�� �������������)
#�������� �������� ��� ������, � ������� DNS ������� ��������� ������ �������� semanage fcontext -a -t FILE_TYPE named.ddns.lab.view1.jnl,
#��� ��������� ������ ���� �� ��������� ����� ��������� ������������ dnssec_trigger_var_run_t, ipa_var_lib_t, krb5_host_rcache_t, krb5_keytab_t,
#named_cache_t, named_log_t, named_tmp_t, named_var_run_t, named_zone_t,
#� ����� ��������� ������ ��������� � ���� restorecon -v named.ddns.lab.view1.jnl. 
#� ������ ������ ��������� ������� ��� ����� named.ddns.lab.view1.jnl, 
#� ������ ���� DNS ������ ���������� ������������ ���������� �� DNS ��������.