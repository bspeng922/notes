yum -y install net-snmp-libs net-snmp net-snmp-utils
systemctl enable snmpd.service
systemctl start snmpd.service