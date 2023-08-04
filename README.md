
# RGW-Multisite Installation Ceph

## Requirements
* Two running Ceph Clusters
* Each cluster must contain 4 Nodes
* Ceph 5 onwards

## Primary Cluster/Zone Configurations

First Create a primary realm.
```bash
radosgw-admin realm create --rgw-realm=my_realm --default
```

Create Primary Zone and Zonegroup.
```bash
radosgw-admin zonegroup create --rgw-zonegroup=us --endpoints=http://10.74.254.43:80 --master --default
```
```bash
radosgw-admin zone create --rgw-zonegroup=us --rgw-zone=us-east-1 --endpoints=http://10.74.254.43:80 --access-key=987654321 --secret-key=123456789
```

Create one User.
```bash
radosgw-admin user create --uid=sysadm --display-name="SysAdmin" --access-key=123456789 --secret=9876546321 --system
```

Create endpoints for primary zone. Provide Primary cluster node ip which do not contain any deployed service (mon, mgr etc).
```bash
radosgw-admin zone modify --rgw-realm=my_realm --rgw-zonegroup=us --rgw-zone=us-east-1 --endpoints http://10.74.254.43:80 --access-key=123456789 --secret=9876546321  --master --default
```

Update the Ceph Configuration Database according to your configuration settings.
```bash
ceph config set client.rgw.test.rgw1.nptdaj rgw_realm my_realm
ceph config set client.rgw.test.rgw1.nptdaj rgw_zonegroup
ceph config set client.rgw.test.rgw1.nptdaj rgw_zone us-east-1
```
Update the Period.
```bash
radosgw-admin period update --commit
```
Deploy the rgw service on respective nodes.
```bash
ceph orch apply rgw test --realm=my_realm --zone=us --placement="2 rgw1 rgw2"
```

check the service using.
```bash
ceph orch ls
```
Restart the rgw services.
```bash
ceph orch restart rgw.test
```

## Secondary Cluster/Zone Configurations.


Pull the primary relam from primary cluster and mention the primary node Ip which you have mentioned previously.

```bash
radosgw-admin realm pull --url=http://10.74.254.43:80 --access-key=123456789 --secret=9876546321
```

Pull the period.
```bash
radosgw-admin period pull --url=http://10.74.254.43:80 --access-key=123456789 --secret=9876546321
```

Create secondary zone in secondary cluster. Make sure to provide secondary cluster any node Ip on which you want to deploy rgw service.
```bash
radosgw-admin zone create --rgw-zonegroup=us --rgw-zone=us-east-2 --endpoints=http://10.74.253.153:80 --access-key=123456789 --secret=9876546321
```
Commit the period.
```bash
radosgw-admin period update --rgw-realm=my_realm --commit
```

Deploy the rgw service on the nodes in the secondary cluster.
```bash
ceph orch apply rgw test --realm=my_realm --zone=test_zone --placement="2 rgw1.example.com rgw2.example.com"
```


Update the Ceph Configuration Database according to your configuration settings.
```bash
ceph config set client.rgw.test.rgw1.nptdaj rgw_realm my_realm
ceph config set client.rgw.test.rgw1.nptdaj rgw_zonegroup us
ceph config set client.rgw.test.rgw1.nptdaj rgw_zone us-east-2
```

Commit the period.
```bash
radosgw-admin period update --rgw-realm=my_realm --commit
```

Get the sync status.
```bash
radosgw-admin sync status
```
