# OSSEC HIDS

## Install from package

Add apt key: 

    wget -q -O - https://www.atomicorp.com/RPM-GPG-KEY.atomicorp.txt  | sudo apt-key add -

Add repo to sources: 

    source /etc/lsb-release
    echo "deb https://updates.atomicorp.com/channels/atomic/ubuntu $DISTRIB_CODENAME main" | sudo tee -a /etc/apt/sources.list.d/atomic.list

    sudo apt update

Install the software

    sudo apt install -y ossec-hids-server

## Install from source

Install prerequisites: 

    sudo apt install gcc make libevent-dev zlib1g-dev  libssl-dev libpcre2-dev wget tar -y

Download and extract source: 

    wget https://github.com/ossec/ossec-hids/archive/3.6.0.tar.gz
    tar xzf 3.6.0.tar.gz
    cd ./ossec-hids-3.6.0

Build and install: 

    sudo ./install.sh

## Configuration

Setup a local mail relay using postfix. 

Example OSSEC config file: 


```xml
<!-- OSSEC example config -->

<ossec_config>
<global>
    <email_notification>yes</email_notification>
    <email_to>alerts@onetwoseven.one</email_to>
    <smtp_server>localhost</smtp_server>
    <email_from>ossec@onetwoseven.one</email_from>
    <email_maxperhour>6</email_maxperhour>
</global>

  <rules>
    <!-- Unused rules have been disabled -->
    <include>rules_config.xml</include>
    <include>pam_rules.xml</include>
    <include>sshd_rules.xml</include>
    <include>telnetd_rules.xml</include>
    <include>syslog_rules.xml</include>
    <include>arpwatch_rules.xml</include>
    <include>named_rules.xml</include>
    <include>smbd_rules.xml</include>
    <include>vsftpd_rules.xml</include>
    <include>wordpress_rules.xml</include>
    <include>web_rules.xml</include>
    <include>web_appsec_rules.xml</include>
    <include>apache_rules.xml</include>
    <include>nginx_rules.xml</include>
    <include>php_rules.xml</include>
    <include>mysql_rules.xml</include>
    <include>postgresql_rules.xml</include>
    <include>ids_rules.xml</include>
    <include>squid_rules.xml</include>
    <include>firewall_rules.xml</include>
    <include>apparmor_rules.xml</include>
    <include>postfix_rules.xml</include>
    <include>sendmail_rules.xml</include>
    <include>imapd_rules.xml</include>
    <include>spamd_rules.xml</include>
    <include>trend-osce_rules.xml</include>
    <!-- <include>policy_rules.xml</include> -->
    <include>ossec_rules.xml</include>
    <include>attack_rules.xml</include>
    <include>dropbear_rules.xml</include>
    <include>unbound_rules.xml</include>
    <include>opensmtpd_rules.xml</include>
    <include>exim_rules.xml</include>
    <include>openbsd-dhcpd_rules.xml</include>
    <include>dnsmasq_rules.xml</include>
    <include>local_rules.xml</include>
  </rules>


  <syscheck>
    <!-- Frequency that syscheck is executed - default every 20 hours -->
    <frequency>72000</frequency>

    <!-- Directories to check  (perform all possible verifications) -->
    <directories report_changes="yes" realtime="yes" check_all="yes">/etc,/usr/bin,/usr/sbin</directories>
    <directories report_changes="yes" realtime="yes" check_all="yes">/bin,/sbin</directories>

    <!-- Files/directories to ignore -->
    <ignore>/etc/mtab</ignore>
    <ignore>/etc/hosts.deny</ignore>
    <ignore>/etc/mail/statistics</ignore>
    <ignore>/etc/random-seed</ignore>
    <ignore>/etc/random.seed</ignore>
    <ignore>/etc/adjtime</ignore>
    <ignore>/etc/httpd/logs</ignore>

    <!-- Check the file, but never compute the diff -->
    <nodiff>/etc/ssl/private/*</nodiff>

    <!-- Ignore files that change frequently--> 
    <auto_ignore>yes</auto_ignore>
    <alert_new_files>yes</alert_new_files>
  </syscheck>

  <rootcheck>
    <rootkit_files>/var/ossec/etc/shared/rootkit_files.txt</rootkit_files>
    <rootkit_trojans>/var/ossec/etc/shared/rootkit_trojans.txt</rootkit_trojans>
  </rootcheck>

  <global>
    <white_list>127.0.0.1</white_list>
    <white_list>::1</white_list>
    <white_list>135.23.236.76</white_list>
  </global>

  <remote>
    <connection>secure</connection>
  </remote>

  <alerts>
    <log_alert_level>3</log_alert_level>
    <email_alert_level>10</email_alert_level>
  </alerts>

  <command>
    <name>host-deny</name>
    <executable>host-deny.sh</executable>
    <expect>srcip</expect>
    <timeout_allowed>yes</timeout_allowed>
  </command>

  <command>
    <name>firewall-drop</name>
    <executable>firewall-drop.sh</executable>
    <expect>srcip</expect>
    <timeout_allowed>yes</timeout_allowed>
  </command>

  <command>
    <name>disable-account</name>
    <executable>disable-account.sh</executable>
    <expect>user</expect>
    <timeout_allowed>yes</timeout_allowed>
  </command>

  <command>
    <name>ossec-slack</name>
    <executable>ossec-slack.sh</executable>
    <expect></expect>
    <timeout_allowed>no</timeout_allowed>
  </command>


  <!-- Active Response Config -->
  <active-response>
    <command>host-deny</command>
    <location>local</location>
    <level>9</level>
    <timeout>3600</timeout>
  </active-response>

  <active-response>
    <command>firewall-drop</command>
    <location>local</location>
    <level>9</level>
    <timeout>3600</timeout>
  </active-response>

  <active-response>
    <command>ossec-slack</command>
    <location>local</location>
    <level>9</level>
  </active-response>

  <!-- Files to monitor (localfiles) -->

  <localfile>
    <log_format>syslog</log_format>
    <location>/var/log/syslog</location>
  </localfile>

  <localfile>
    <log_format>syslog</log_format>
    <location>/var/log/auth.log</location>
  </localfile>

  <localfile>
    <log_format>syslog</log_format>
    <location>/var/log/secure</location>
  </localfile>

  <localfile>
    <log_format>syslog</log_format>
    <location>/var/log/mail.log</location>
  </localfile>

  <localfile>
    <log_format>apache</log_format>
    <location>/var/log/nginx/access.log</location>
  </localfile>

  <localfile>
    <log_format>apache</log_format>
    <location>/var/log/nginx/error.log</location>
  </localfile>

  <localfile>
    <log_format>syslog</log_format>
    <location>/var/log/kern.log</location>
  </localfile>

  <!-- Check suricata log file -->
  <localfile>
    <log_format>snort-fast</log_format>
    <location>/var/log/suricata/fast.log</location>
  </localfile>

  <!-- Check active responses -->
  <localfile>
    <log_format>syslog</log_format>
    <location>/var/ossec/logs/active-responses.log</location>
  </localfile>

  <!-- Check disk space and high load --> 
  <localfile>
    <log_format>command</log_format>
    <command>df -h</command>
  </localfile>

  <localfile>
    <log_format>command</log_format>
    <command>uptime</command>
  </localfile>

</ossec_config>

```