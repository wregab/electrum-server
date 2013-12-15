IRC is used by LTCLectrum server to find 'peers' - other LTCLectrum servers. The
current list can be seen by running:

    ./server.py peers

The following config file options are used by the IRC part of LTCLectrum server:

    [server]
    irc = yes
    host = fqdn.host.name.tld
    #report_host = fqdn.host.name.tld
    irc_nick = server nickname
    #report_stratum_http_port = 80

`irc` is used to determine whether the IRC thread will be started or the 
LTCLectrum server will run in private mode. In private mode, 
`./server.py peers` will always return an empty list.

`host` is a fully-qualified domain name (FQDN) of your LTCLectrum server. It is
used both when binding the listener for incoming client connections and as part
of the realname field in IRC (see below).

`report_host` is a an optional fully-qualified domain name (FQDN) of your LTCLectrum server 
instead of `host`. It is used as part of the name field in IRC for incoming client connections.
This is useful in a NAT setup where you bind to a private IP locally but have an external IP
set up at your router and external DNS.

`report_stratum_tcp_port`, `report_stratum_http_port`, `report_stratum_tcp_ssl_port`, 
`report_stratum_http_ssl_port` are optional settings for a port number to be reported in the
IRC name field without actually binding this port locally. This is useful in a NAT setup
where you might want to bind to a high port locally but DNAT a different possibly privileged
port for inbound connections

`irc_nick` is a nick name that will be appended to the L_ suffix when 
composing the IRC nickname to identify your server on #ltclectrum.

Please note the IRC name field can only contain 50 chars and will be composed
of `host` + protocol version number + Port numbers for the various protocols.
Please check whether port numbers are cut off at the end   

