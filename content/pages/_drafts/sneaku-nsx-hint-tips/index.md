---
title: "SneakU NSX Hint & Tips"
date: 2015-03-16
draft: true
---

\[expand title="Slow Login to vCenter/NSX Plugin"\]

When building NSX, after SSO integration with AD, test speed of web interface with a Domain User who is a member of more than 5 AD groups ([https://communities.vmware.com/message/2478606#2478606](https://communities.vmware.com/message/2478606#2478606))

Changes were made to 6.1.4 which has made login process HEAPS quicker. LDAP lookups for memberOf are now done in parallel and perform some caching.

\[/expand\]

* * *

\[expand title="DRS Rules/Groups for Controllers/ESG/DLR"\]

Before you decide to re-deploy an ESG, make sure you take note of any DRS configured. Upon re-deployment, it appears as though the DRS Rules are modified or the VM removed from the DRS rules, which means they may not be enforcing DRS as planned.

\[/expand\]

* * *

\[expand title="Check HA status of ESG or DLR"\]

```
dlr-nsx-01.gw.sydney.edu.au-0> show service highavailability 
Highavailability Status:             running
Highavailability Unit Name:          dlr-nsx-01.gw.sydney.edu.au-0
Highavailability Unit State:         active
Highavailability Interface(s):       vNic_0 
Unit Poll Policy:
   Frequency:                      3     seconds
   Deadtime:                       15    seconds
   Stateful Sync-up Time:          10    seconds
Highavailability Healthcheck Status:
   Peer host [dlr-nsx-01.gw.sydney.edu.au-1]: good
   This host [dlr-nsx-01.gw.sydney.edu.au-0]: good
Highavailability Stateful Logical Status:
   File-Sync                       running
   Connection-Sync                 running
      xmit       xerr       rcv        rerr      
      11020484   0          10994648   0
```

\[/expand\]

* * *

\[expand title="Cisco ECMP Commands"\] On Cisco Catalyst 6500 Series switches, when used as the upstream router with ECMP, use the following commands to identify which ESG a particular flow is going through

```
sh ip route vrf transit 10.83.64.0
```

```
Routing Table: transit
Routing entry for 10.83.64.0/24
  Known via "ospf 9", distance 110, metric 1, type NSSA extern 2, forward metric 102
  Redistributing via bgp 65297
  Advertised by bgp 65297 route-map ospf9-to-bgp
  Last update from 172.21.141.252 on Vlan3584, 1d20h ago
  Routing Descriptor Blocks:
    172.21.141.252, from 10.83.2.254, 1d20h ago, via Vlan3584
      Route metric is 1, traffic share count is 1
  * 172.21.141.251, from 10.83.2.254, 1d20h ago, via Vlan3584
      Route metric is 1, traffic share count is 1

```

CEF is enabled so need to check the CEF table to verify the next hop for that particular flow/subnet. By default CEF will do flow based distribution.

```
sh ip cef vrf transit 10.83.64.0 detail 

```

```
10.83.64.0/24, epoch 2, per-destination sharing
  nexthop 172.21.141.251 Vlan3584
  nexthop 172.21.141.252 Vlan3584

```

When CEF is enabled, to show the exact path a flow takes, run the following command

```
show ip cef vrf transit exact-route 10.165.16.70 10.83.64.240 dest-port 22

```

```
10.165.16.70 -> 10.83.64.240 => IP adj out of Vlan3584, addr 172.21.141.251
```

\[/expand\]

* * *

\[expand title="Why can't I ping/connect to Protocol Address of DLR Control VM when connected to a logical switch"\]

This behaviour is as expected due to the fact that the control VM has the subnet assigned to the logical switch configured on the internal VDR interface on the control VM. This means that the return packet will never reach the VM connected to the logical switch.

\[/expand\]

* * *

\[expand title="Custom filtering on DFW CLI commands"\]

```
~ # summarize-dvfilter | grep -A 1 'vmm\|sfw\.2'
world 36933 vmm0:NSX-Test-03 vcUuid:'50 1d a5 36 1c b2 b7 40-b5 3b 44 7d 3c be b1 e7'
 port 100663306 NSX-Test-03.eth0
--
   name: nic-36933-eth0-vmware-sfw.2
   agentName: vmware-sfw
--
world 3818771 vmm0:NSX-Test-NTP vcUuid:'50 1d 3b 8d ac 5d 4d 0d-9a 91 67 85 da e8 01 53'
 port 100663307 NSX-Test-NTP.eth0
--
   name: nic-3818771-eth0-vmware-sfw.2
   agentName: vmware-sfw
   
   
~ # vsipioctl getfwrules -f  nic-35662-eth0-vmware-sfw.2 -s
rule  1004: 317802 evals, in 0 out 0 pkts, in 0 out 0 bytes
rule  1004: 0 evals, in 0 out 0 pkts, in 0 out 0 bytes
rule  1003: 317802 evals, in 0 out 0 pkts, in 0 out 0 bytes
rule  1003: 0 evals, in 0 out 0 pkts, in 0 out 0 bytes
rule  1002: 317802 evals, in 2959418 out 3212647 pkts, in 648188117 out 719943929 bytes
rule  1001: 8 evals, in 2961052 out 3214274 pkts, in 652908589 out 723246387 bytes
 
Let’s filter out on “in != 0”:
 
~ # vsipioctl getfwrules -f  nic-35662-eth0-vmware-sfw.2 -s | awk '{ if ($6 != "0") print; }'
rule  1002: 317912 evals, in 2960565 out 3213857 pkts, in 648405157 out 720318479 bytes
rule  1001: 8 evals, in 2962199 out 3215484 pkts, in 653128551 out 723622917 bytes

```

\[/expand\]

* * *

\[expand title="Follow the MAC/IP Address"\]

Commands to run when trying to find out which host and VM/IP address resides on

```
nsx-controller # show control-cluster logical-switches arp-table 5014
VNI      IP              MAC               Connection-ID
5014     10.83.64.1      00:50:56:9d:17:e1 33           

nsx-controller # show control-cluster logical-switches vtep-table 5014
VNI      IP              Segment         MAC               Connection-ID
5014     10.83.253.3     10.83.253.0     00:50:56:67:34:4b 38           
5014     10.83.253.16    10.83.253.0     00:50:56:69:fa:be 33           
5014     10.83.253.10    10.83.253.0     00:50:56:64:36:6d 33           
5014     10.83.253.9     10.83.253.0     00:50:56:63:4d:a0 34           
5014     10.83.253.14    10.83.253.0     00:50:56:6f:5f:1d 34           

nsx-controller # show control-cluster logical-switches mac-table 5014
VNI      MAC               VTEP-IP         Connection-ID
5014     00:50:56:9d:0a:f2 10.83.253.14    34           
5014     00:50:56:9d:17:e1 10.83.253.16    33           

nsx-controller # show control-cluster logical-switches vtep-table 5014
VNI      IP              Segment         MAC               Connection-ID
5014     10.83.253.3     10.83.253.0     00:50:56:67:34:4b 38           
5014     10.83.253.16    10.83.253.0     00:50:56:69:fa:be 33           
5014     10.83.253.10    10.83.253.0     00:50:56:64:36:6d 33           
5014     10.83.253.9     10.83.253.0     00:50:56:63:4d:a0 34           
5014     10.83.253.14    10.83.253.0     00:50:56:6f:5f:1d 34           

nsx-controller # show control-cluster logical-switches vni 5014
VNI      Controller      BUM-Replication ARP-Proxy Connections VTEPs
5014     10.85.183.222   Enabled         Enabled   7           5    

```

\[/expand\]

* * *

 

\[expand title="\* List all the NSX Database Tables"\] The NSX Manager runs a Postgresql DB. This is the DB table layout.

```
secureall=# \dt;
                                  List of relations
 Schema |                          Name                          | Type  |   Owner   
--------+--------------------------------------------------------+-------+-----------
 public | acauditlog_x                                           | table | secureall
 public | acceptedcvgtrial                                       | table | secureall
 public | access_control_entry                                   | table | secureall
 public | acuser_x                                               | table | secureall
 public | acuser_x_userroles_x                                   | table | secureall
 public | acuserresource_x                                       | table | secureall
 public | acuserrole_x                                           | table | secureall
 public | ai_action_info                                         | table | secureall
 public | ai_action_info_dest_security_group                     | table | secureall
 public | ai_action_info_src_security_group                      | table | secureall
 public | ai_app                                                 | table | secureall
 public | ai_attribute_configurable                              | table | secureall
 public | ai_blacklistips                                        | table | secureall
 public | ai_blacklistusers                                      | table | secureall
 public | ai_dbattr                                              | table | secureall
 public | ai_dbmaintain_item                                     | table | secureall
 public | ai_dbmaintain_rec                                      | table | secureall
 public | ai_desktop_pool                                        | table | secureall
 public | ai_domain                                              | table | secureall
 public | ai_ec_attribute                                        | table | secureall
 public | ai_ec_policy_action                                    | table | secureall
 public | ai_eventlogserver                                      | table | secureall
 public | ai_group                                               | table | secureall
 public | ai_group_do                                            | table | secureall
 public | ai_group_member                                        | table | secureall
 public | ai_grouptouserflat                                     | table | secureall
 public | ai_host                                                | table | secureall
 public | ai_hostipmap                                           | table | secureall
 public | ai_ldapserver                                          | table | secureall
 public | ai_listener_info                                       | table | secureall
 public | ai_orgunit                                             | table | secureall
 public | ai_port_description                                    | table | secureall
 public | ai_security_group                                      | table | secureall
 public | ai_syslog_config                                       | table | secureall
 public | ai_user                                                | table | secureall
 public | ai_user_do                                             | table | secureall
 public | ai_useripmap                                           | table | secureall
 public | ai_usertogroupflat                                     | table | secureall
 public | ai_vm_agent                                            | table | secureall
 public | ai_vmipmap                                             | table | secureall
 public | alerttag                                               | table | secureall
 public | app_datacenter_state                                   | table | secureall
 public | app_firewall_config                                    | table | secureall
 public | app_firewall_failed_publish_info                       | table | secureall
 public | app_firewall_rules                                     | table | secureall
 public | app_fwconfig_fwrule_mapping_layer2                     | table | secureall
 public | app_fwconfig_fwrule_mapping_layer3                     | table | secureall
 public | audit_logs                                             | table | secureall
 public | auditalert                                             | table | secureall
 public | backupprefs_x                                          | table | secureall
 public | ca_certificate_store                                   | table | secureall
 public | certificate_store                                      | table | secureall
 public | client_topics                                          | table | secureall
 public | cm_config                                              | table | secureall
 public | crl_store                                              | table | secureall
 public | csr_store                                              | table | secureall
 public | decoderalert_x                                         | table | secureall
 public | dependent_task                                         | table | secureall
 public | deployment_container                                   | table | secureall
 public | deployment_container_compute                           | table | secureall
 public | deployment_container_storage                           | table | secureall
 public | deployment_unit                                        | table | secureall
 public | dhcp_ip_pools                                          | table | secureall
 public | dhcp_log_info                                          | table | secureall
 public | dhcp_static_bindings                                   | table | secureall
 public | dlp_classification                                     | table | secureall
 public | dlp_classification_value                               | table | secureall
 public | dlp_completed_scan_summary                             | table | secureall
 public | dlp_policy                                             | table | secureall
 public | dlp_policy_excluded_areas                              | table | secureall
 public | dlp_policy_excluded_security_groups                    | table | secureall
 public | dlp_policy_included_security_groups                    | table | secureall
 public | dlp_policy_regulation                                  | table | secureall
 public | dlp_regulation                                         | table | secureall
 public | dlp_regulation_classification                          | table | secureall
 public | dlp_violating_file                                     | table | secureall
 public | dlp_violation_info                                     | table | secureall
 public | domain_object                                          | table | secureall
 public | domain_object_extattr                                  | table | secureall
 public | domain_object_info                                     | table | secureall
 public | domain_object_ipelements                               | table | secureall
 public | domain_object_macelements                              | table | secureall
 public | domain_object_relationships                            | table | secureall
 public | domain_object_transport_elements                       | table | secureall
 public | edge                                                   | table | secureall
 public | edge_allocated_ip_address                              | table | secureall
 public | edge_appliance                                         | table | secureall
 public | edge_appliance_custom_fields                           | table | secureall
 public | edge_appliance_interfaces                              | table | secureall
 public | edge_appliance_resource_allocations                    | table | secureall
 public | edge_appliance_vmx_params                              | table | secureall
 public | edge_bgp_filters                                       | table | secureall
 public | edge_bgp_neighbours                                    | table | secureall
 public | edge_bgp_redistribution_rules                          | table | secureall
 public | edge_bgp_routing_config                                | table | secureall
 public | edge_bridges                                           | table | secureall
 public | edge_certificate                                       | table | secureall
 public | edge_cli_credentials                                   | table | secureall
 public | edge_config                                            | table | secureall
 public | edge_current_features                                  | table | secureall
 public | edge_dhcp_config_ip_pools                              | table | secureall
 public | edge_dhcp_config_relay                                 | table | secureall
 public | edge_dhcp_config_relay_agent                           | table | secureall
 public | edge_dhcp_config_static_bindings                       | table | secureall
 public | edge_dhcp_relay_config_data                            | table | secureall
 public | edge_dns_client                                        | table | secureall
 public | edge_dns_records                                       | table | secureall
 public | edge_dns_views                                         | table | secureall
 public | edge_dns_zones                                         | table | secureall
 public | edge_edge_vnic_lport_uuid_map                          | table | secureall
 public | edge_firewall_default_policy                           | table | secureall
 public | edge_firewall_rules                                    | table | secureall
 public | edge_flow_stats                                        | table | secureall
 public | edge_fw_config                                         | table | secureall
 public | edge_fw_rules                                          | table | secureall
 public | edge_global_service_instance                           | table | secureall
 public | edge_gslb_global_config                                | table | secureall
 public | edge_gslb_global_ip                                    | table | secureall
 public | edge_gslb_global_ip_pool_ref                           | table | secureall
 public | edge_gslb_global_site                                  | table | secureall
 public | edge_gslb_global_site_server                           | table | secureall
 public | edge_gslb_listen_on_ip                                 | table | secureall
 public | edge_gslb_monitor                                      | table | secureall
 public | edge_gslb_pool                                         | table | secureall
 public | edge_gslb_pool_member                                  | table | secureall
 public | edge_gslb_pool_monitor_id                              | table | secureall
 public | edge_gslb_security                                     | table | secureall
 public | edge_gslb_security_ca_certificates                     | table | secureall
 public | edge_gslb_security_crl_certificates                    | table | secureall
 public | edge_ha_zero_conf_block                                | table | secureall
 public | edge_health_status                                     | table | secureall
 public | edge_interface_stats                                   | table | secureall
 public | edge_internal_arp_filter_interface_policy              | table | secureall
 public | edge_internal_arp_filter_rules                         | table | secureall
 public | edge_internal_grouping_objects                         | table | secureall
 public | edge_internal_nat_rules                                | table | secureall
 public | edge_internal_static_routes                            | table | secureall
 public | edge_ipsec_config_site_config                          | table | secureall
 public | edge_ipsec_config_site_map                             | table | secureall
 public | edge_ipsec_global_ca_certificate_config                | table | secureall
 public | edge_ipsec_global_crl_certificate_config               | table | secureall
 public | edge_isis_interfaces                                   | table | secureall
 public | edge_isis_redistribution_rules                         | table | secureall
 public | edge_isis_routing_config                               | table | secureall
 public | edge_l2vpn_client_filters                              | table | secureall
 public | edge_l2vpn_config                                      | table | secureall
 public | edge_l2vpn_config_peer_site                            | table | secureall
 public | edge_l2vpn_config_proxy_setting                        | table | secureall
 public | edge_l2vpn_config_user                                 | table | secureall
 public | edge_l2vpn_encryption_algorithms                       | table | secureall
 public | edge_l2vpn_peer_site_filters                           | table | secureall
 public | edge_l2vpn_peer_site_stretched_vnics                   | table | secureall
 public | edge_load_balancer_application_profile                 | table | secureall
 public | edge_load_balancer_application_rule                    | table | secureall
 public | edge_load_balancer_application_rule_id                 | table | secureall
 public | edge_load_balancer_attribute                           | table | secureall
 public | edge_load_balancer_client_ssl                          | table | secureall
 public | edge_load_balancer_client_ssl_ca_certificate           | table | secureall
 public | edge_load_balancer_client_ssl_crl_certificate          | table | secureall
 public | edge_load_balancer_client_ssl_server_certificate       | table | secureall
 public | edge_load_balancer_healthcheck                         | table | secureall
 public | edge_load_balancer_member                              | table | secureall
 public | edge_load_balancer_member_healthcheck                  | table | secureall
 public | edge_load_balancer_member_service_port                 | table | secureall
 public | edge_load_balancer_member_v4                           | table | secureall
 public | edge_load_balancer_monitor                             | table | secureall
 public | edge_load_balancer_pool                                | table | secureall
 public | edge_load_balancer_pool_application_rule_id            | table | secureall
 public | edge_load_balancer_pool_monitor_id                     | table | secureall
 public | edge_load_balancer_pool_v4                             | table | secureall
 public | edge_load_balancer_profile_attributes                  | table | secureall
 public | edge_load_balancer_server_ssl                          | table | secureall
 public | edge_load_balancer_server_ssl_ca_certificate           | table | secureall
 public | edge_load_balancer_server_ssl_client_certificate       | table | secureall
 public | edge_load_balancer_server_ssl_crl_certificate          | table | secureall
 public | edge_load_balancer_service_port                        | table | secureall
 public | edge_load_balancer_service_profile                     | table | secureall
 public | edge_load_balancer_vendor_profile                      | table | secureall
 public | edge_load_balancer_virtual_server                      | table | secureall
 public | edge_load_balancer_virtual_server_v4                   | table | secureall
 public | edge_nat_rules                                         | table | secureall
 public | edge_ospf_areas                                        | table | secureall
 public | edge_ospf_interfaces                                   | table | secureall
 public | edge_ospf_redistribution_rules                         | table | secureall
 public | edge_ospf_routing_config                               | table | secureall
 public | edge_publish_state                                     | table | secureall
 public | edge_query_daemon                                      | table | secureall
 public | edge_routing_config                                    | table | secureall
 public | edge_routing_ip_prefixes                               | table | secureall
 public | edge_service_config                                    | table | secureall
 public | edge_service_config_default_policy_rules               | table | secureall
 public | edge_service_config_internal_rules                     | table | secureall
 public | edge_service_config_user_rules                         | table | secureall
 public | edge_services_startup_type                             | table | secureall
 public | edge_si_attribute                                      | table | secureall
 public | edge_si_attributes                                     | table | secureall
 public | edge_si_service_instance_runtime_nic_info              | table | secureall
 public | edge_si_service_profile_ven_tables                     | table | secureall
 public | edge_si_typed_attribute                                | table | secureall
 public | edge_si_typed_attribute_table                          | table | secureall
 public | edge_si_typed_attribute_table_rows                     | table | secureall
 public | edge_si_typed_attributes                               | table | secureall
 public | edge_sslvpn_config                                     | table | secureall
 public | edge_sslvpn_config_advanced_configuration              | table | secureall
 public | edge_sslvpn_config_auth_config                         | table | secureall
 public | edge_sslvpn_config_auth_servers                        | table | secureall
 public | edge_sslvpn_config_client_configuration                | table | secureall
 public | edge_sslvpn_config_client_icon                         | table | secureall
 public | edge_sslvpn_config_client_install_package_gateway_list | table | secureall
 public | edge_sslvpn_config_client_install_packages             | table | secureall
 public | edge_sslvpn_config_file_data                           | table | secureall
 public | edge_sslvpn_config_ippools                             | table | secureall
 public | edge_sslvpn_config_layout                              | table | secureall
 public | edge_sslvpn_config_private_networks                    | table | secureall
 public | edge_sslvpn_config_script                              | table | secureall
 public | edge_sslvpn_config_script_file                         | table | secureall
 public | edge_sslvpn_config_server_settings                     | table | secureall
 public | edge_sslvpn_config_users                               | table | secureall
 public | edge_sslvpn_config_web_resources                       | table | secureall
 public | edge_sslvpn_server_settings_cipher_list                | table | secureall
 public | edge_sslvpn_server_settings_ssl_version_list           | table | secureall
 public | edge_static_routes                                     | table | secureall
 public | edge_system_control_property                           | table | secureall
 public | edge_traffic_stats                                     | table | secureall
 public | edge_version_based_configs                             | table | secureall
 public | edge_version_map                                       | table | secureall
 public | edge_vm                                                | table | secureall
 public | edge_vm_custom_fields                                  | table | secureall
 public | edge_vm_info                                           | table | secureall
 public | edge_vnic                                              | table | secureall
 public | edge_vnic_address_group                                | table | secureall
 public | edge_vnic_fence_params                                 | table | secureall
 public | edge_vnic_group                                        | table | secureall
 public | edge_vnic_mac_address_map_forhavms                     | table | secureall
 public | enappcmismatchinfo_x                                   | table | secureall
 public | enappcsyslogconfig_x                                   | table | secureall
 public | enappcsystemconfig_x                                   | table | secureall
 public | enappliance_x                                          | table | secureall
 public | enclustergwstate                                       | table | secureall
 public | encustomerinfo_x                                       | table | secureall
 public | endnsservers_x                                         | table | secureall
 public | endpoint_global_scan_info                              | table | secureall
 public | endpoint_policy_action                                 | table | secureall
 public | endpoint_scan_status                                   | table | secureall
 public | endpoint_security_manager_excluded                     | table | secureall
 public | endpoint_security_manager_included                     | table | secureall
 public | endpoint_security_manager_policy                       | table | secureall
 public | endpoint_security_solution_registration                | table | secureall
 public | endpoint_security_svm_active                           | table | secureall
 public | endpoint_security_vendor_registration                  | table | secureall
 public | endpoint_vm_scaninfo                                   | table | secureall
 public | englobaltrapdestination_x                              | table | secureall
 public | enidsequence_x                                         | table | secureall
 public | enldapserver_x                                         | table | secureall
 public | ensmtpserver_x                                         | table | secureall
 public | entrapdestination_x                                    | table | secureall
 public | entrustedcertificate_x                                 | table | secureall
 public | esxhostinfo                                            | table | secureall
 public | event_code                                             | table | secureall
 public | extended_attribute                                     | table | secureall
 public | extended_attribute_meta                                | table | secureall
 public | firewall_appliance                                     | table | secureall
 public | firewall_config                                        | table | secureall
 public | firewall_draft                                         | table | secureall
 public | firewall_policy_action                                 | table | secureall
 public | firewall_rule                                          | table | secureall
 public | firewall_rule_applied_to_list                          | table | secureall
 public | firewall_rule_destinations                             | table | secureall
 public | firewall_rule_services                                 | table | secureall
 public | firewall_rule_siprofile                                | table | secureall
 public | firewall_rule_sources                                  | table | secureall
 public | firewall_ruleid_sequence                               | table | secureall
 public | firewall_section                                       | table | secureall
 public | firewall_si_rule_set                                   | table | secureall
 public | firewall_state                                         | table | secureall
 public | firewall_status_cluster                                | table | secureall
 public | firewall_status_host                                   | table | secureall
 public | flow_ipfix_collector                                   | table | secureall
 public | flow_ipfix_config                                      | table | secureall
 public | flows_active                                           | table | secureall
 public | flows_aggregation_record                               | table | secureall
 public | flows_all                                              | table | secureall
 public | flows_vnics                                            | table | secureall
 public | flowstats                                              | table | secureall
 public | flowstats_l2                                           | table | secureall
 public | flowstats_l3                                           | table | secureall
 public | flowstats_rollup                                       | table | secureall
 public | global_config_defaults                                 | table | secureall
 public | gwperformancestatus_x                                  | table | secureall
 public | hint_question_answer                                   | table | secureall
 public | host_install_info                                      | table | secureall
 public | housekeeping_module                                    | table | secureall
 public | id_sequencer                                           | table | secureall
 public | ip_addresses                                           | table | secureall
 public | ip_addresses_ip_addresses_strings                      | table | secureall
 public | ip_namespace_info                                      | table | secureall
 public | ipam_allocated_ip_address                              | table | secureall
 public | ipam_ip_range                                          | table | secureall
 public | ipam_subnet                                            | table | secureall
 public | ipelement                                              | table | secureall
 public | ipsec_tunnel_events                                    | table | secureall
 public | ipsec_vpn_global_config                                | table | secureall
 public | ipsec_vpn_ike_stats                                    | table | secureall
 public | ipsec_vpn_site_stats                                   | table | secureall
 public | ipsec_vpn_sites_config                                 | table | secureall
 public | ipsec_vpn_tunnel_stats                                 | table | secureall
 public | job_data                                               | table | secureall
 public | job_data_task_dependency_map                           | table | secureall
 public | job_instance                                           | table | secureall
 public | job_instance_job_output                                | table | secureall
 public | job_instance_task_instances                            | table | secureall
 public | job_schedule                                           | table | secureall
 public | key_value_store                                        | table | secureall
 public | l2vpn_client_stretched_vnics                           | table | secureall
 public | lb_backend_servers                                     | table | secureall
 public | load_balancer_listeners                                | table | secureall
 public | macelement                                             | table | secureall
 public | messaging_client                                       | table | secureall
 public | messaging_system_account                               | table | secureall
 public | nat_rules                                              | table | secureall
 public | nvs_logical_switch                                     | table | secureall
 public | nvs_logical_switch_zone_mapping                        | table | secureall
 public | nvs_nvp_node                                           | table | secureall
 public | nvs_transport_node                                     | table | secureall
 public | nvs_transport_node_zone_mapping                        | table | secureall
 public | nwfabric_feature_info                                  | table | secureall
 public | nwfabric_feature_info_included_features                | table | secureall
 public | nwfabric_feature_info_required_features                | table | secureall
 public | nwfabric_resource_status_feature_status                | table | secureall
 public | qualyslastupdatelog                                    | table | secureall
 public | qualysserverconfig                                     | table | secureall
 public | qualysupdatelog                                        | table | secureall
 public | realmid_mappings                                       | table | secureall
 public | regulation_categories                                  | table | secureall
 public | regulation_regions                                     | table | secureall
 public | relationship_extattr                                   | table | secureall
 public | reportpreferences_x                                    | table | secureall
 public | rflx_class_mapping                                     | table | secureall
 public | route_config                                           | table | secureall
 public | sc_reference_mappings                                  | table | secureall
 public | scompatibilitymatrixentry_x                            | table | secureall
 public | server_settings_cipher_list                            | table | secureall
 public | si_attribute                                           | table | secureall
 public | si_attributes                                          | table | secureall
 public | si_category                                            | table | secureall
 public | si_functionality                                       | table | secureall
 public | si_implementation                                      | table | secureall
 public | si_profile_runtime_dvpgs                               | table | secureall
 public | si_profile_runtime_vwires                              | table | secureall
 public | si_runtime_vlan_mapping                                | table | secureall
 public | si_service                                             | table | secureall
 public | si_service_depends_on                                  | table | secureall
 public | si_service_deploy_spec                                 | table | secureall
 public | si_service_deploy_spec_scope                           | table | secureall
 public | si_service_functionalities                             | table | secureall
 public | si_service_implementations                             | table | secureall
 public | si_service_instance                                    | table | secureall
 public | si_service_instance_bridge_edge                        | table | secureall
 public | si_service_instance_config                             | table | secureall
 public | si_service_instance_runtime_clusters                   | table | secureall
 public | si_service_instance_runtime_data_networks              | table | secureall
 public | si_service_instance_runtime_dep_scope                  | table | secureall
 public | si_service_instance_runtime_info                       | table | secureall
 public | si_service_instance_runtime_nicinfo                    | table | secureall
 public | si_service_instance_template                           | table | secureall
 public | si_service_manager                                     | table | secureall
 public | si_service_profile                                     | table | secureall
 public | si_service_profile_binding_status                      | table | secureall
 public | si_service_profile_dvpgs                               | table | secureall
 public | si_service_profile_excluded_vnics                      | table | secureall
 public | si_service_profile_rules                               | table | secureall
 public | si_service_profile_security_groups                     | table | secureall
 public | si_service_profile_ven_secs                            | table | secureall
 public | si_service_profile_ven_tables                          | table | secureall
 public | si_service_profile_virtual_servers                     | table | secureall
 public | si_service_profile_virtual_wires                       | table | secureall
 public | si_service_transports                                  | table | secureall
 public | si_service_used_by                                     | table | secureall
 public | si_service_vendor_preference                           | table | secureall
 public | si_stateful_ip_address                                 | table | secureall
 public | si_stateful_ip_port                                    | table | secureall
 public | si_stateful_mac                                        | table | secureall
 public | si_stateful_policy                                     | table | secureall
 public | si_stateful_qualifier                                  | table | secureall
 public | si_stateful_rule                                       | table | secureall
 public | si_stateful_rule_ip_containers                         | table | secureall
 public | si_stateful_rule_ip_port_containers                    | table | secureall
 public | si_stateful_rule_ip_port_set                           | table | secureall
 public | si_stateful_rule_ip_set                                | table | secureall
 public | si_stateful_rule_mac_containers                        | table | secureall
 public | si_stateful_rule_mac_set                               | table | secureall
 public | si_stateful_ruleset                                    | table | secureall
 public | si_tcp_option                                          | table | secureall
 public | si_transport                                           | table | secureall
 public | si_typed_attribute                                     | table | secureall
 public | si_typed_attribute_table                               | table | secureall
 public | si_typed_attribute_table_rows                          | table | secureall
 public | si_typed_attributes                                    | table | secureall
 public | si_vendor_section                                      | table | secureall
 public | si_vendor_section_tables                               | table | secureall
 public | si_vendor_template                                     | table | secureall
 public | si_vendor_template_availabilty_zones                   | table | secureall
 public | si_vendor_template_functionalities                     | table | secureall
 public | si_vendor_template_tables                              | table | secureall
 public | si_vendor_template_ven_secs                            | table | secureall
 public | si_versioned_deploy_spec                               | table | secureall
 public | sit_ports                                              | table | secureall
 public | sitappinfo_service                                     | table | secureall
 public | sitapplparameter                                       | table | secureall
 public | sitapplparameter_values                                | table | secureall
 public | sitappmodule                                           | table | secureall
 public | sitconflict                                            | table | secureall
 public | sitdosparameter                                        | table | secureall
 public | sitentry                                               | table | secureall
 public | sitentry_applparameter                                 | table | secureall
 public | sitentry_appmodules                                    | table | secureall
 public | sitentry_apppatchdisable                               | table | secureall
 public | sitentry_apppatchenable                                | table | secureall
 public | sitentry_apppatches                                    | table | secureall
 public | sitentry_disabledservices                              | table | secureall
 public | sitentry_dosparameter                                  | table | secureall
 public | sitentry_expparameter                                  | table | secureall
 public | sitexpparameter                                        | table | secureall
 public | sitexpparameter_values                                 | table | secureall
 public | sitservice                                             | table | secureall
 public | spoof_guard_info                                       | table | secureall
 public | spoof_guard_info_ip_address                            | table | secureall
 public | spoof_guard_policy                                     | table | secureall
 public | spoof_guard_policy_context_ids                         | table | secureall
 public | spoof_guard_setting                                    | table | secureall
 public | spoofguard_ip_details                                  | table | secureall
 public | spoofguard_setting                                     | table | secureall
 public | ssoconfig                                              | table | secureall
 public | ssystemversionentry_x                                  | table | secureall
 public | stdataelement_x                                        | table | secureall
 public | stranded_svms                                          | table | secureall
 public | stranded_svms_metadata                                 | table | secureall
 public | sttimestamp_x                                          | table | secureall
 public | svm_instance_info                                      | table | secureall
 public | svradvisoryentry                                       | table | secureall
 public | svrtrafficstats                                        | table | secureall
 public | syslog_server_config                                   | table | secureall
 public | system_event_event_metadata                            | table | secureall
 public | system_event_message_params                            | table | secureall
 public | system_events                                          | table | secureall
 public | systemalarm                                            | table | secureall
 public | task                                                   | table | secureall
 public | task_dependency                                        | table | secureall
 public | task_dependency_tasks                                  | table | secureall
 public | task_instance                                          | table | secureall
 public | task_instance_task_data                                | table | secureall
 public | task_instance_task_output                              | table | secureall
 public | task_model_base                                        | table | secureall
 public | task_policy                                            | table | secureall
 public | task_target                                            | table | secureall
 public | task_task_init_data                                    | table | secureall
 public | traffic_steering_policy_action                         | table | secureall
 public | transport_element                                      | table | secureall
 public | uaactionitem_x                                         | table | secureall
 public | uaccessrule_x                                          | table | secureall
 public | uaglobalactionitemprefs_x                              | table | secureall
 public | uaglobalactionitemprefs_x_en_z                         | table | secureall
 public | uaglobalactionitemprefs_x_pu_z                         | table | secureall
 public | uaidsequence_x                                         | table | secureall
 public | uappliancestate_x                                      | table | secureall
 public | uappliancestate_x_accessrule_z                         | table | secureall
 public | uapplication_x                                         | table | secureall
 public | uapplication_x_applcntxtpara_z                         | table | secureall
 public | uapplication_x_decenhancemen_z                         | table | secureall
 public | uapplication_x_disdecelement_z                         | table | secureall
 public | uapplication_x_disvwps_x                               | table | secureall
 public | uapplication_x_dosparams_x                             | table | secureall
 public | uapplication_x_enabledvuls_x                           | table | secureall
 public | uapplication_x_endecelements_x                         | table | secureall
 public | uapplication_x_expcntrlparam_z                         | table | secureall
 public | uapplication_x_vfixes_x                                | table | secureall
 public | uapplparameter_x                                       | table | secureall
 public | uapplparameter_x_values_x                              | table | secureall
 public | uaproxysettings_x                                      | table | secureall
 public | uapublisher_x                                          | table | secureall
 public | uapullinterval_x                                       | table | secureall
 public | uareleaseactionitem_x                                  | table | secureall
 public | ubasedecoder_x                                         | table | secureall
 public | udecodingenhancement_x                                 | table | secureall
 public | udisableddecodeelement_x                               | table | secureall
 public | udosparameter_x                                        | table | secureall
 public | udz_nodes                                              | table | secureall
 public | uenableddecodeelement_x                                | table | secureall
 public | uexpparameter_x                                        | table | secureall
 public | uexpparameter_x_values_x                               | table | secureall
 public | unique_id                                              | table | secureall
 public | uosinfo_x                                              | table | secureall
 public | user_defined_zones                                     | table | secureall
 public | user_password_hint                                     | table | secureall
 public | user_role_mappings                                     | table | secureall
 public | useremailalertprefs_x                                  | table | secureall
 public | userinfo                                               | table | secureall
 public | userrptpref_x                                          | table | secureall
 public | userviceprofile_x                                      | table | secureall
 public | userviceprofile_x_appliances_z                         | table | secureall
 public | userviceprofile_x_dproxyserv_z                         | table | secureall
 public | userviceprofile_x_selectedpo_z                         | table | secureall
 public | usvcgroupprofile_x                                     | table | secureall
 public | usvcgroupprofile_x_appliedpa_z                         | table | secureall
 public | usvcgroupprofile_x_applversi_z                         | table | secureall
 public | usvcgroupprofile_x_dissubmod_z                         | table | secureall
 public | usvcgroupprofile_x_disvwps_x                           | table | secureall
 public | usvcgroupprofile_x_osdata_x                            | table | secureall
 public | usvcgroupprofile_x_patchdisa_z                         | table | secureall
 public | usvcgroupprofile_x_patchenab_z                         | table | secureall
 public | usvcgroupprofile_x_svcprofil_z                         | table | secureall
 public | uvirtualfix_x                                          | table | secureall
 public | uvirtualfix_x_enabledvuls_x                            | table | secureall
 public | uvulnerability_x                                       | table | secureall
 public | vapplication                                           | table | secureall
 public | vc_inventory_serialized                                | table | secureall
 public | vdc_root_info                                          | table | secureall
 public | vdn_cluster                                            | table | secureall
 public | vdn_cluster_vds_contexts                               | table | secureall
 public | vdn_scope                                              | table | secureall
 public | vdn_scope_vdn_clusters                                 | table | secureall
 public | vdn_vds_context                                        | table | secureall
 public | vdn_virtual_wire                                       | table | secureall
 public | vdn_virtual_wire_backing                               | table | secureall
 public | vdn_vmknic_portgroup                                   | table | secureall
 public | vds_teaming_uplink_port                                | table | secureall
 public | vib_install_info                                       | table | secureall
 public | vim_compute_resource_host_ids                          | table | secureall
 public | vim_compute_resource_network_ids                       | table | secureall
 public | vim_datacenter_datastore_ids                           | table | secureall
 public | vim_distributed_virtual_switch_dv_portgroups_ids       | table | secureall
 public | vim_distributed_virtual_switch_host_member             | table | secureall
 public | vim_distributed_virtual_switch_uplink_names            | table | secureall
 public | vim_distributed_virtual_switch_uplink_pg_ids           | table | secureall
 public | vim_host_system_datastore_ids                          | table | secureall
 public | vim_host_system_network_ids                            | table | secureall
 public | vim_host_system_nic_info                               | table | secureall
 public | vim_host_system_opaque_network_ids                     | table | secureall
 public | vim_host_system_vm_ids                                 | table | secureall
 public | vim_network_host_ids                                   | table | secureall
 public | vim_object_property                                    | table | secureall
 public | vim_virtual_machine_datastore_ids                      | table | secureall
 public | vim_vmware_distributed_virtual_switch_lacpv2lag_names  | table | secureall
 public | vim_vnic_ip_address_meta                               | table | secureall
 public | vim_vnic_published_ip_address_meta                     | table | secureall
 public | vmwarevirtualcenterconfig_x                            | table | secureall
 public | vnic_group_vnic_names                                  | table | secureall
 public | vnvp_controller                                        | table | secureall
 public | vnvp_host_key                                          | table | secureall
 public | vnvp_vdr_instance                                      | table | secureall
 public | vnvp_vdr_instance_backing                              | table | secureall
 public | vpn_client_configuration                               | table | secureall
 public | vpn_server_settings                                    | table | secureall
 public | vpn_web_resource                                       | table | secureall
 public | vserver                                                | table | secureall
 public | vserverstatus                                          | table | secureall
 public | vservice                                               | table | secureall
 public | vsidsequence_x                                         | table | secureall
 public | vsm_configuration                                      | table | secureall
 public | vsm_global_parameters                                  | table | secureall
 public | vsm_upgrade_info                                       | table | secureall
 public | vsmagent                                               | table | secureall
 public | vulnerabilityviolation_x                               | table | secureall
 public | xvs_cluster_info                                       | table | secureall
 public | xvs_connectivity_test_history                          | table | secureall
 public | xvs_host_info                                          | table | secureall
 public | xvs_network                                            | table | secureall
 public | xvs_network_feature                                    | table | secureall
 public | xvs_nsm                                                | table | secureall
 public | xvs_range                                              | table | secureall
 public | xvs_resource                                           | table | secureall
 public | xvs_resource_def                                       | table | secureall
 public | xvs_switch                                             | table | secureall
 public | xvs_vdn_scope_test                                     | table | secureall
 public | xvs_vdn_segment                                        | table | secureall
 public | xvs_vmknic_info                                        | table | secureall
(576 rows)

secureall=#
```

\[/expand\]

* * *

\[expand title="NSX Manager Engineering Mode"\]

To access the engineering shell on the NSX Manager

```
st eng
IAmOnThePhone..)
psql -U secureall
```

Lots of config files live in the following directory

```
/home/secureall/secureall/sem/WEB-INF/spring
```

\[/expand\]

* * *

\[expand title="NSX Manager PostGres Commands"\]

The following will show which hosts and which cluster are meant to have DFW rules pushed down to them. This was taken from a customer where it was trying to push DFW rules to a host which wasn't in a NSX prepared cluster.

```
secureall=# select * from firewall_status_host;
 host_id | cluster_id  | error_code | latest_generation_number | current_generation_number |  start_time   |   end_time    
---------+-------------+------------+--------------------------+---------------------------+---------------+---------------
 host-71 | domain-c26  |            |            1425451691257 |             1425451691257 | 1425451859659 | 1425451859687
 host-31 | domain-c26  |            |            1424919237179 |             1425451691257 | 1425347827749 | 1425451859942
 host-29 | domain-c26  |            |            1425451691257 |             1425447088341 | 1425451860054 | 1425447093972
 host-35 | domain-c26  |            |            1425451691257 |             1425447088341 | 1425451860166 | 1425447093976
 host-44 | domain-c603 |            |            1425451691257 |             1425451691257 | 1425451860206 | 1425451860203
 host-51 | domain-c603 |            |            1425451691257 |             1425451691257 | 1425451860273 | 1425451860280
 host-54 | domain-c41  |     301501 |            1425380449044 |                           | 1425424873657 | 1425434504202
 host-45 | domain-c603 |            |            1425451691257 |             1425447088341 | 1425451860280 | 1425447094005
 host-43 | domain-c603 |            |            1425451691257 |             1425447088341 | 1425451860371 | 1425447094007
(9 rows)

secureall=# select * from firewall_status_cluster;
 cluster_id  | generation_number 
-------------+-------------------
 domain-c41  |     1425432139307
 domain-c26  |     1425451691257
 domain-c603 |     1425451691257
(3 rows)
```

Delete the offending entries from the database, refresh the UI and the errors should disappear.

```
for each unexpected host listed in firewall_status_host table:
  - delete from firewall_status_host where host_id='host-XXX';

for each unexpected cluster listed in firewall_status_cluster table:
  - delete from firewall_status_cluster where cluster_id='domain-YYY';
```

\[/expand\]

* * *

List all NICs on a host

```
~ # esxcli network nic list 
Name    PCI Device     Driver  Link  Speed  Duplex  MAC Address         MTU  Description                                                   
------ ------------- ------ ---- ----- ------ ----------------- ---- --------------------------------------------------------------
vmnic0  0000:001:00.0  bnx2x   Up    10000  Full    f8:db:88:e9:98:af  1500  Broadcom Corporation NetXtreme II BCM57840 10 Gigabit Ethernet
vmnic1  0000:001:00.1  bnx2x   Up    10000  Full    f8:db:88:e9:98:b2  1500  Broadcom Corporation NetXtreme II BCM57840 10 Gigabit Ethernet
vmnic2  0000:001:00.2  bnx2x   Up    10000  Full    f8:db:88:e9:9d:7f  1500  Broadcom Corporation NetXtreme II BCM57840 10 Gigabit Ethernet
vmnic3  0000:001:00.3  bnx2x   Up    10000  Full    f8:db:88:e9:9d:82  1500  Broadcom Corporation NetXtreme II BCM57840 10 Gigabit Ethernet
vmnic4  0000:003:00.0  bnx2x   Up    10000  Full    f8:db:88:e9:98:b3  9000  Broadcom Corporation NetXtreme II BCM57810 10 Gigabit Ethernet
vmnic5  0000:003:00.1  bnx2x   Up    10000  Full    f8:db:88:e9:98:b6  9000  Broadcom Corporation NetXtreme II BCM57810 10 Gigabit Ethernet

```

Get NIC driver version

```
~ # ethtool -i vmnic0
driver: bnx2x
version: 2.710.70.v55.7
firmware-version: FFV7.10.17 bc 7.10.11
bus-info: 0000:01:00.0

```

Another way

```
~ # vmkload_mod -s bnx2x | grep Version
 Version: Version 2.710.70.v55.7, Build: 1331820, Interface: 9.2 Built on: Nov 10 2014

```

Get more details about a specific NIC. This also includes driver version

```
~ # esxcli network nic get -n vmnic2
   Advertised Auto Negotiation: true
   Advertised Link Modes: 1000baseT/Full, 10000baseT/Full
   Auto Negotiation: true
   Cable Type: FIBRE
   Current Message Level: 0
   Driver Info: 
         Bus Info: 0000:01:00.2
         Driver: bnx2x
         Firmware Version: FFV7.10.18 bc 7.10.11
         Version: 2.710.39.v55.2
   Link Detected: true
   Link Status: Up 
   Name: vmnic2
   PHYAddress: 1
   Pause Autonegotiate: true
   Pause RX: true
   Pause TX: true
   Supported Ports: FIBRE
   Supports Auto Negotiation: true
   Supports Pause: true
   Supports Wakeon: true
   Transceiver: internal
   Wakeon: MagicPacket(tm)
```

Check current options configured on NIC driver - In this instance its the Broadcom bnx2x driver

```
~ # esxcfg-module -g bnx2x
bnx2x enabled = 1 options = ''
```

Enable RSS on ESXi Broadcom Driver

```
~ # esxcfg-module -s 'RSS=4' bnx2x
~ # esxcfg-module -g bnx2x
bnx2x enabled = 1 options = 'RSS=4'
```

Check if RSS enabled on physical NIC

```
~ # vsish -e get /net/pNics/vmnic2/rxqueues/info
rx queues info {
   # queues supported:8
   # filters supported:132
   # active filters:0
   Rx Queue features:features: 0x200001a3 -> LRO Pair Dynamic RSS Dynamic Preemptible IPv4LRO
}
```

You can also verify that at this point multiple queues are created per physical NIC.

```
~ # ethtool -S vmnic2 | grep rx| grep ucast
     [0]: rx_ucast_packets: 105403
     [1]: rx_ucast_packets: 0
     [2]: rx_ucast_packets: 0
     [3]: rx_ucast_packets: 0
     [4]: rx_ucast_packets: 940146
     [5]: rx_ucast_packets: 593595
     [6]: rx_ucast_packets: 613160
     [7]: rx_ucast_packets: 630126
     rx_ucast_packets: 2882430
```

Once VTEPS have been created, do some basic pings

```
ping ++netstack=vxlan -d -s 1572 -I vmk4 10.83.253.2
```

Run packet capture on other end to make sure traffic is received, you can also see packet size etc.

```
pktcap-uw --uplink vmnic3 -dir 0 --srcip 10.83.253.4
The name of the uplink is vmnic3
The dir is Rx
The session filter source IP address is 10.83.253.2
No server port specifed, select 57681 as the port
Output the packet info to console.
Local CID 2
Listen on port 57681
Accept...Vsock connection from port 1028 cid 2
23:52:02.839002[1] Captured at EtherswitchDispath point, TSO not enabled, Checksum not offloaded and not verified, VLAN tag 4053, length 1614.
        Segment[0] ---- 1614 bytes:
        0x0000:  0050 566f 860d 0050 5666 0c74 0800 4500 
        0x0010:  0640 edb5 4000 4001 3856 0a53 fd02 0a53 
        0x0020:  fd08 0800 c5ad bc00 0000 54ef b1a2 000c 
<output truncated>
```

Use following link to check for TSO and LRO [http://kb.vmware.com/selfservice/microsites/search.do?language=en\_US&cmd=displayKC&externalId=2055140](http://kb.vmware.com/selfservice/microsites/search.do?language=en_US&cmd=displayKC&externalId=2055140)

Use the following link to check for RSS on ixgbe NIC driver [http://kb.vmware.com/selfservice/microsites/search.do?language=en\_US&cmd=displayKC&externalId=2034676](http://kb.vmware.com/selfservice/microsites/search.do?language=en_US&cmd=displayKC&externalId=2034676)

Useful link about RSS Netqueues [http://kb.vmware.com/selfservice/microsites/search.do?cmd=displayKC&docType=kc&externalId=2032810&sliceId=1&docTypeID=DT\_KB\_1\_1&dialogID=541071144&stateId=1%200%20526448381](http://kb.vmware.com/selfservice/microsites/search.do?cmd=displayKC&docType=kc&externalId=2032810&sliceId=1&docTypeID=DT_KB_1_1&dialogID=541071144&stateId=1%200%20526448381)

On ESXi host, list DLR instances

```
~ # net-vdr -I -l

VDR Instance Information :
---------------------------

Vdr Name:                   default+edge-1
Vdr Id:                     1460487509
Number of Lifs:             16
Number of Routes:           17
State:                      Enabled
Controller IP:              10.85.183.222
Control Plane IP:           10.85.83.11
Control Plane Active:       Yes
Num unique nexthops:        1
Generation Number:          0
Edge Active:                No
```

 

! Find all logical router instances on controllers

```
nsx-controller # show control-cluster logical-router instance all
LR-Id      LR-Name            Hosts[]         Edge-Connection Service-Controller
0x570d4555 default+edge-1                                     10.85.183.222
```

Show interface summary on logical router, similar to show ip int brief

```
nsx-controller # show control-cluster logical-routers interface-summary 0x570d4555
Interface                        Type   Id           IP[]              
570d455500000015                 vxlan  0x1394       10.83.48.254/24   
570d455500000012                 vxlan  0x1391       10.83.38.254/24   
570d45550000000b                 vxlan  0x138a       10.83.21.254/24   
570d455500000011                 vxlan  0x1390       10.83.36.254/24   
570d45550000000e                 vxlan  0x138d       10.83.31.254/24   
570d455500000018                 vxlan  0x1397       10.83.65.254/24   
570d455500000016                 vxlan  0x1395       10.83.49.254/24   
570d455500000013                 vxlan  0x1392       10.83.40.254/24   
570d45550000000f                 vxlan  0x138e       10.83.32.254/24   
570d455500000010                 vxlan  0x138f       10.83.34.254/24   
570d455500000002                 vxlan  0x1388       10.83.2.254/28    
570d45550000000c                 vxlan  0x138b       10.83.22.254/24   
570d455500000017                 vxlan  0x1396       10.83.64.254/24   
570d45550000000a                 vxlan  0x1389       10.83.20.254/24   
570d45550000000d                 vxlan  0x138c       10.83.30.254/24   
570d455500000014                 vxlan  0x1393       10.83.41.254/24
```

ESXi - show DLR routing table

```
~ # net-vdr -R -l default+edge-1

VDR default+edge-1 Route Table
Legend: [U: Up], [G: Gateway], [C: Connected], [I: Interface]
Legend: [H: Host], [F: Soft Flush] [!: Reject] [E: ECMP]

Destination      GenMask          Gateway          Flags    Ref Origin   UpTime     Interface
----------- ------- ------- ----- --- ------ ------ ---------
10.83.2.240      255.255.255.240  0.0.0.0          UCI      1   MANUAL   250361     570d455500000002
10.83.20.0       255.255.255.0    0.0.0.0          UCI      1   MANUAL   250361     570d45550000000a
10.83.21.0       255.255.255.0    0.0.0.0          UCI      1   MANUAL   250360     570d45550000000b
10.83.22.0       255.255.255.0    0.0.0.0          UCI      1   MANUAL   250360     570d45550000000c
10.83.30.0       255.255.255.0    0.0.0.0          UCI      1   MANUAL   750        570d45550000000d
10.83.31.0       255.255.255.0    0.0.0.0          UCI      1   MANUAL   723        570d45550000000e
10.83.32.0       255.255.255.0    0.0.0.0          UCI      1   MANUAL   696        570d45550000000f
10.83.34.0       255.255.255.0    0.0.0.0          UCI      1   MANUAL   667        570d455500000010
10.83.36.0       255.255.255.0    0.0.0.0          UCI      1   MANUAL   639        570d455500000011
10.83.38.0       255.255.255.0    0.0.0.0          UCI      1   MANUAL   610        570d455500000012
10.83.40.0       255.255.255.0    0.0.0.0          UCI      1   MANUAL   578        570d455500000013
10.83.41.0       255.255.255.0    0.0.0.0          UCI      1   MANUAL   549        570d455500000014
10.83.48.0       255.255.255.0    0.0.0.0          UCI      1   MANUAL   523        570d455500000015
10.83.49.0       255.255.255.0    0.0.0.0          UCI      1   MANUAL   497        570d455500000016
10.83.64.0       255.255.255.0    0.0.0.0          UCI      1   MANUAL   465        570d455500000017
10.83.65.0       255.255.255.0    0.0.0.0          UCI      1   MANUAL   441        570d455500000018
10.83.254.0      255.255.255.0    10.83.2.242      UG       1   AUTO     192648     570d455500000002
```

Check if NSX Controller is listening for SSH

```
nsx-controller # show service cli listen-address

0.0.0.0:22
```
