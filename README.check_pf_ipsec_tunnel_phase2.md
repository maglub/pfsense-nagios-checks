# Introduction

In some environments, you might have a number of phase 2 connections. One common issue is that one side does not remove old connections after re-keying.

This check will count the number of phase 2 connections for a tuple of subnets.

Basically, you get the same information by running:

```
ipsec statusall
```

# Examples

## Icinga2 config

This is an example of an icinga2 configuration:

```
apply Service "IPSEC to Customer X - Phase 2 192.168.1.0/24 === 10.0.0.0/24" {
  import "generic-service"
  check_command = "by_ssh"

  #--- IPSEC config
  vars.ipsec_endpoint      = "130.241.142.10"
  vars.ipsec_p2_source     = "192.168.1.0/24"
  vars.ipsec_p2_dest       = "10.0.0.0/24"
  vars.ipsec_display_name  = "Customer X"

  #--- firewall address
  vars.by_ssh_address      = "172.16.20.21"

  vars.by_ssh_command = [ "/usr/local/bin/sudo", "/opt/icinga-plugins/custom/pfsense-nagios-checks/check_pf_ipsec_tunnel_phase2"]
  vars.by_ssh_logname = "icinga2"
  vars.by_ssh_arguments = {
    "--endpoint" = "$ipsec_endpoint$"
    "--p2-source" = "$ipsec_p2_source$"
    "--p2-dest" = "$ipsec_p2_dest$"
    "--display-name" = "$ipsec_display_name$"
  }

  notes_url = "https://ip-of-pfsense-firewall/status_ipsec.php"
  assign where ( host.address && host.display_name in [ "some-icinga2-host"] )
}
```

You will also need to do the following:

* Put the scripts on your pfsense firewall under /opt/icinga-plugins/custom/pfsense-nagios-checks/
* Create an icinga2 user on your pfsense firewall
* Install sudo on your firewall (System => Package Manager)
* Configure sudo so that the icinga2 user can execute commands without password (System => sudo)
