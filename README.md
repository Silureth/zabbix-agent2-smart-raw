# zabbix-agent2-smart-raw
zabbix-agent2 with raw data , precompiled on DEBIAN 10 
 - go version go1.22.3 linux/amd64
 - libc6 (2.28-10+deb10u3)
# ZIPPED CHANGES to smart plugin in smart.zip
src/go/plugins/smart
# PRECOMPILED agent is based on
zabbix_agent2 (Zabbix) 6.0.31rc1
# zbx_export_templates.yaml
for LTS 6.0, included changes for "normalized values"
MACROS for it
{$SMART.WEAR_LEVEL.CRIT}
{$SMART.WEAR_LEVEL.WARN}
# USED
changes made by this pull:
https://github.com/zabbix/zabbix/pull/95
# BUILD INSTRUCTIONS
https://medium.com/@zhuravlev.vitaly/writing-watcher-zabbix-agent2-mqtt-plugin-in-go-e00067ea1990