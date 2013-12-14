<!---
$"metadata"$
{"md": true, "upload_date": "2009-11-27 14:13:54", "title": "Rabbitmq and iptables", "draft": false, "slug": "rabbitmq-and-iptables", "tags": ["rabbitmq", "iptables"]}
$"metadata"$
-->
FYI:

If you anytime want to install rabbitmq in a machine running iptables, please, remember to ACCEPT everything comming from localhost (at least the ports needed by rabbitmq and erlang).

### To add to the iptables.rules file

-I INPUT -s 127.0.0.1 -p tcp -j ACCEPT
-I INPUT -s 127.0.0.1 -p udp -j ACCEPT

Bye