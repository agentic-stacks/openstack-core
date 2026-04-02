# Diagnose — Symptom-Based Troubleshooting Decision Trees

Use this skill when an operator reports a problem and you need to systematically narrow down the cause. Each section follows the pattern: symptom → diagnostic commands → likely causes → resolution.

**Before any diagnosis**: confirm which release is running (`openstack versions show` or `openstack --version`) and collect the output of `openstack endpoint list` so you know the service topology.

---

## Instance Won't Launch

**Symptom**: `openstack server create` returns an error, or the server reaches ERROR state.

### Step 1 — Check nova-scheduler logs

```bash
# SystemD deployments
journalctl -u devstack@n-sch --since "10 minutes ago" | grep -i "error\|no valid host\|filter"

# Package deployments
tail -200 /var/log/nova/nova-scheduler.log | grep -i "error\|no valid host\|filter\|NoValidHost"
```

Look for `NoValidHost`, filter names that rejected all hosts, or placement errors. The log line includes which filters eliminated hosts.

### Step 2 — Check Placement allocation candidates

```bash
# List allocation candidates for a flavor
openstack allocation candidate list \
  --resource VCPU=2,MEMORY_MB=4096,DISK_GB=40

# If the list is empty, no host can satisfy the request
# Check each compute node's inventory
openstack resource provider list
openstack resource provider inventory list <RESOURCE_PROVIDER_UUID>

# Check existing allocations on a provider
openstack resource provider allocation list <RESOURCE_PROVIDER_UUID>

# Show usages
openstack resource provider usage show <RESOURCE_PROVIDER_UUID>
```

Empty candidate list means either no capacity or a trait/aggregate mismatch.

### Step 3 — Check compute node capacity

```bash
# Hypervisor summary
openstack hypervisor stats show

# Per-hypervisor detail
openstack hypervisor list --long
openstack hypervisor show <HYPERVISOR_HOSTNAME>

# Check for disabled/down compute services
openstack compute service list
openstack compute service list --service nova-compute
```

If a compute service shows `down`, the nova-compute process on that node is not running or not connecting to RabbitMQ. If `disabled`, it was manually disabled.

```bash
# On the affected compute node — check nova-compute
systemctl status devstack@n-cpu   # DevStack
systemctl status nova-compute     # Package
journalctl -u nova-compute --since "10 minutes ago"
```

### Step 4 — Check Neutron port binding

```bash
# After a failed launch, find the port
openstack port list --server <SERVER_ID>

# Show port details — look at binding:vif_type and binding:vnic_type
openstack port show <PORT_ID>
```

If `binding:vif_type` is `binding_failed`, the ML2 mechanism driver could not bind the port to the host.

```bash
# Check Neutron agents on the target compute node
openstack network agent list --host <COMPUTE_HOSTNAME>

# Check Neutron server logs
journalctl -u devstack@q-svc --since "10 minutes ago" | grep -i "port\|bind\|error"
tail -200 /var/log/neutron/neutron-server.log | grep -i "bind\|error"
```

Common causes: L2 agent is down on the compute node, network not reachable from that host, physnet mapping mismatch.

### Step 5 — Check Glance image availability

```bash
openstack image show <IMAGE_ID>
# Verify status = active, size > 0, visibility allows the project
```

If status is `queued` or `killed`, the image upload failed or the Glance store backend is unavailable.

```bash
# Check Glance API logs
journalctl -u devstack@g-api --since "10 minutes ago"
tail -100 /var/log/glance/glance-api.log | grep -i "error"

# Verify the image file actually exists in the store
openstack image show <IMAGE_ID> -f json | jq '.["direct_url"]'
```

### Step 6 — Check Cinder volume (boot-from-volume only)

```bash
openstack volume show <VOLUME_ID>
# Must be status=available (or in-use for attach-existing)

# Check Cinder logs
journalctl -u devstack@c-api --since "10 minutes ago"
tail -100 /var/log/cinder/cinder-api.log | grep -i "error"
```

### Common Resolutions

| Cause | Resolution |
|---|---|
| NoValidHost / no candidates | Free capacity, fix traits, check aggregate membership |
| Compute service down | Restart nova-compute, fix RabbitMQ connectivity |
| Port binding failed | Restart L2 agent on compute node, fix physnet mapping |
| Image not active | Re-upload image, check Glance backend |
| Over quota | Increase quota (`openstack quota set`) or delete unused resources |
| Scheduler filter rejecting all hosts | Review `NoValidHost` log lines to identify which filter; adjust flavor/image metadata or host aggregate |

---

## Instance Has No Network

**Symptom**: Instance boots but cannot reach the network — no DHCP response, no ping, no connectivity.

### Step 1 — Check security groups

```bash
# List security group rules for the server's port
openstack port show <PORT_ID>
# Note the security_group_ids

openstack security group rule list <SECURITY_GROUP_ID>
# Verify ingress/egress rules allow expected traffic
# Default security group blocks all ingress from outside the group
```

Add required rules:

```bash
openstack security group rule create --protocol icmp --ingress <SG_ID>
openstack security group rule create --protocol tcp --dst-port 22 --ingress <SG_ID>
```

### Step 2 — Check port status

```bash
openstack port show <PORT_ID>
# status must be ACTIVE
# binding:vif_type must NOT be binding_failed
# binding:host_id must match the compute node
```

If `status = DOWN` after the instance is ACTIVE, the L2 agent has not plugged the port.

### Step 3 — Check DHCP agent

```bash
# List agents servicing the network
openstack network agent list --network <NETWORK_ID>

# Verify DHCP agent is alive and active
openstack network agent show <DHCP_AGENT_ID>

# Check DHCP namespace on the network node
# Find the namespace: ip netns list | grep qdhcp
sudo ip netns list | grep qdhcp-<NETWORK_ID_PREFIX>

# Verify dnsmasq is running inside the namespace
sudo ip netns exec qdhcp-<NETWORK_ID> ps aux | grep dnsmasq

# Check if the DHCP port exists
openstack port list --network <NETWORK_ID> --device-owner network:dhcp
```

DHCP agent logs:

```bash
journalctl -u devstack@q-dhcp --since "10 minutes ago"
tail -100 /var/log/neutron/neutron-dhcp-agent.log | grep -i "error\|<NETWORK_ID_SHORT>"
```

### Step 4 — Check L2 agent on the compute node

```bash
# OVS deployments
openstack network agent list --host <COMPUTE_HOST> --agent-type "Open vSwitch agent"

# OVN deployments
openstack network agent list --host <COMPUTE_HOST> --agent-type "OVN Controller agent"

# Linux Bridge deployments
openstack network agent list --host <COMPUTE_HOST> --agent-type "Linux bridge agent"
```

If the agent is down, SSH to the compute node and check:

```bash
# OVS
systemctl status neutron-openvswitch-agent
journalctl -u neutron-openvswitch-agent --since "10 minutes ago"

# OVN
systemctl status ovn-controller
ovn-appctl -t ovn-controller connection-status

# Linux Bridge
systemctl status neutron-linuxbridge-agent
```

### Step 5 — Check bridge and OVS configuration

```bash
# On the compute node — OVS
sudo ovs-vsctl show
# Verify the integration bridge (br-int) exists
# Verify the physical bridge (br-ex, br-provider) exists and has the physical NIC
# Verify the patch ports between br-int and br-ex

# Check the tap device for the instance is on br-int
sudo ovs-vsctl list-ports br-int | grep tap

# OVN — check logical port status
sudo ovn-nbctl show | grep -A5 "<PORT_ID_PREFIX>"
sudo ovn-sbctl show | grep -A5 "<PORT_ID_PREFIX>"

# Linux Bridge
brctl show
ip link show
```

### Step 6 — Check MTU mismatch

```bash
# Check network MTU setting
openstack network show <NETWORK_ID> | grep mtu

# Check instance MTU (inside instance or via console)
openstack console log show <SERVER_ID> | grep -i mtu
# Or: ip link show eth0 (inside instance)

# Check physical NIC MTU on compute node
ip link show <PHYSICAL_NIC>

# Check OVS bridge MTU
ovs-vsctl list interface br-int | grep mtu
```

MTU mismatch causes intermittent packet loss or inability to reach larger MTU destinations. Fix: set network MTU to match physical fabric minus encapsulation overhead (VXLAN: subtract 50, GRE: subtract 42).

```bash
openstack network set --mtu <CORRECT_MTU> <NETWORK_ID>
```

### Common Resolutions

| Cause | Resolution |
|---|---|
| Security group blocks traffic | Add correct ingress/egress rules |
| DHCP agent down | Restart neutron-dhcp-agent, reschedule network |
| L2 agent down on compute | Restart OVS/OVN/LinuxBridge agent |
| tap device not bridged | Restart nova-compute and L2 agent |
| MTU mismatch | Set correct MTU on Neutron network |
| No DHCP namespace | `openstack network agent add network <DHCP_AGENT_ID> <NETWORK_ID>` |

---

## API Returns 401 Unauthorized

**Symptom**: OpenStack CLI or API calls return `401 Unauthorized` or `The request you have made requires authentication`.

### Step 1 — Check token validity

```bash
# Issue a new token and inspect it
openstack token issue
# If this fails, credentials or Keystone endpoint is the problem

# Check token expiry
openstack token issue -f json | jq '.expires'

# Verify environment variables
env | grep OS_
# Check OS_AUTH_URL, OS_USERNAME, OS_PASSWORD, OS_PROJECT_NAME, OS_DOMAIN_NAME
```

### Step 2 — Check endpoint catalog

```bash
openstack catalog list
# Verify the service endpoints are correct and reachable
openstack endpoint list
openstack endpoint list --service identity

# Test connectivity to Keystone
curl -v <OS_AUTH_URL>/v3
```

### Step 3 — Check keystone_authtoken configuration

Every service that validates tokens has a `[keystone_authtoken]` section:

```bash
# Nova
grep -A 20 '\[keystone_authtoken\]' /etc/nova/nova.conf

# Neutron
grep -A 20 '\[keystone_authtoken\]' /etc/neutron/neutron.conf

# Cinder
grep -A 20 '\[keystone_authtoken\]' /etc/cinder/cinder.conf
```

Verify `auth_url`, `username`, `password`, `project_name`, and domain settings. Confirm the service user exists:

```bash
openstack user show nova
openstack user show neutron
openstack role assignment list --user nova --project service
```

### Step 4 — Check Keystone service logs

```bash
journalctl -u devstack@keystone --since "10 minutes ago" | grep -i "error\|401\|invalid"
tail -100 /var/log/keystone/keystone.log | grep -i "error\|401"

# Apache-fronted Keystone
tail -100 /var/log/apache2/keystone_error.log
tail -100 /var/log/httpd/keystone_error.log
```

### Step 5 — Check policy files

```bash
# Locate and check policy files
ls /etc/nova/policy.yaml /etc/nova/policy.json 2>/dev/null
ls /etc/neutron/policy.yaml 2>/dev/null

# Validate a specific rule
oslopolicy-checker \
  --policy /etc/nova/policy.yaml \
  --rule "os_compute_api:servers:create" \
  --target project_id=<PROJECT_ID> \
  --credentials user_id=<USER_ID>,roles=member
```

### Common Resolutions

| Cause | Resolution |
|---|---|
| Token expired | Re-authenticate (`openstack token issue`) |
| Wrong credentials | Fix `OS_PASSWORD`, `OS_USERNAME` in RC file |
| Service user missing | Create service user and assign `service` role |
| Wrong auth_url | Fix `[keystone_authtoken] auth_url` in service config |
| Policy denying access | Review policy.yaml, check role assignments |
| Clock skew > 5 minutes | Sync NTP on all nodes |

---

## API Returns 403 Forbidden

**Symptom**: API calls return `403 Forbidden` — authentication succeeds but authorization fails.

### Step 1 — Identify the policy rule being checked

```bash
# The 403 response body usually names the rule
# Example: "Policy doesn't allow os_compute_api:servers:create to be performed"

# Locate the policy file
ls /etc/<service>/policy.yaml
```

### Step 2 — Check role assignments

```bash
# List roles for the user in the project
openstack role assignment list \
  --user <USER_ID_OR_NAME> \
  --project <PROJECT_ID_OR_NAME> \
  --names

# List all roles in the system
openstack role list

# Check for system-scoped roles (admin operations often need this)
openstack role assignment list --user <USER> --system all --names
```

### Step 3 — Validate with oslopolicy-checker

```bash
oslopolicy-checker \
  --policy /etc/nova/policy.yaml \
  --rule "os_compute_api:os-hypervisors:list" \
  --credentials roles=admin,project_id=<PROJECT_ID>
```

### Common Resolutions

| Cause | Resolution |
|---|---|
| Missing role | `openstack role add --user X --project Y <ROLE>` |
| Default policy too restrictive | Review and customize policy.yaml for the required role |
| Scope mismatch (project vs system) | Assign system-scoped admin role for admin operations |
| Deprecated policy rules | Upgrade policy.yaml to match the release defaults |

---

## API Returns 500 Internal Server Error

**Symptom**: API calls return `500 Internal Server Error`.

### Step 1 — Read service logs immediately

```bash
# Identify which service returned the 500
# Check that service's API log

# Nova
journalctl -u devstack@n-api --since "5 minutes ago" | grep -i "error\|traceback\|exception"
tail -200 /var/log/nova/nova-api.log | grep -i "error\|traceback"

# Neutron
journalctl -u devstack@q-svc --since "5 minutes ago" | grep -i "error\|traceback"
tail -200 /var/log/neutron/neutron-server.log

# Cinder
tail -200 /var/log/cinder/cinder-api.log | grep -i "error\|traceback"

# Glance
tail -200 /var/log/glance/glance-api.log | grep -i "error\|traceback"

# Keystone (often Apache)
tail -200 /var/log/apache2/error.log | grep -i "keystone"
```

Python tracebacks in the log pinpoint the exact failure.

### Step 2 — Check database connectivity

```bash
# Test MySQL/MariaDB connection
mysql -u <SERVICE_USER> -p<PASSWORD> -h <DB_HOST> <SERVICE_DB> -e "SELECT 1"

# Check connection pool from service config
grep -E "connection|pool" /etc/<service>/<service>.conf | grep -i "database\|sql"

# Check for too many connections
mysql -u root -p -e "SHOW STATUS LIKE 'Threads_connected';"
mysql -u root -p -e "SHOW VARIABLES LIKE 'max_connections';"
```

### Step 3 — Check RabbitMQ connectivity

Services that use RPC (Nova, Neutron, Cinder) fail with 500 when they cannot reach RabbitMQ:

```bash
# Check RabbitMQ is running
rabbitmqctl status

# Check service config for transport_url
grep "transport_url" /etc/<service>/<service>.conf

# Test connectivity from the service host
nc -zv <RABBITMQ_HOST> 5672

# Check for memory or disk alarms that block publishing
rabbitmqctl cluster_status | grep -A5 "alarms"
```

### Step 4 — Check configuration syntax

```bash
# Nova
nova-manage config validate 2>&1 | head -50

# Cinder
cinder-manage config validate 2>&1 | head -50

# For any service — check for missing required options
grep -i "error" /var/log/<service>/<service>-api.log | grep -i "config\|option\|unknown"
```

### Step 5 — Run DB migration check

```bash
nova-manage db version
nova-manage api_db version
neutron-db-manage current
cinder-manage db version
```

If the current version does not match the expected version for the release, a migration was not run.

```bash
# Run pending migrations (after backup)
nova-manage db sync
nova-manage api_db sync
neutron-db-manage upgrade heads
cinder-manage db sync
```

### Common Resolutions

| Cause | Resolution |
|---|---|
| DB connectivity failure | Fix `[database] connection` in service config, restart DB |
| DB migration not run | Run `<service>-manage db sync` after backing up DB |
| RabbitMQ unreachable | Fix `transport_url`, restart RabbitMQ |
| Config syntax error | Fix the offending option, restart service |
| Service ran out of workers | Increase `[DEFAULT] api_workers` |

---

## Volume Attach Fails

**Symptom**: `openstack server add volume` fails, or the volume stays in `attaching` state.

### Step 1 — Check Cinder volume status

```bash
openstack volume show <VOLUME_ID>
# status must be 'available' for a fresh attach
# If status is 'error', 'in-use', or 'attaching' (stuck), investigate further

# Check volume is not already attached
openstack volume show <VOLUME_ID> -f json | jq '.attachments'
```

### Step 2 — Check Nova instance status

```bash
openstack server show <SERVER_ID>
# Instance must be ACTIVE or SHUTOFF for attach
# SHELVED_OFFLOADED, SUSPENDED, MIGRATING — attachment may fail
```

### Step 3 — Check target protocol

**iSCSI:**

```bash
# On the compute node where the instance runs
iscsiadm -m session
# Should show the target if discovery succeeded

# Check iSCSI initiator
cat /etc/iscsi/initiatorname.iscsi

# Manual discovery (replace with your Cinder target IP)
iscsiadm -m discovery -t sendtargets -p <CINDER_TARGET_IP>:3260

# Check nova-compute logs for iSCSI errors
journalctl -u nova-compute --since "10 minutes ago" | grep -i "iscsi\|volume\|attach\|error"
```

**Ceph RBD:**

```bash
# Verify RBD access from the compute node
rbd ls <CINDER_POOL>
rbd info <CINDER_POOL>/<VOLUME_ID>

# Check ceph auth
ceph auth get client.cinder
ceph auth get client.nova

# Test RBD map
sudo rbd map <CINDER_POOL>/<VOLUME_ID> --id cinder --keyring /etc/ceph/ceph.client.cinder.keyring
```

**NFS:**

```bash
# Verify NFS mount
df -h | grep nfs
mount | grep nfs

# Check NFS share is accessible
showmount -e <NFS_SERVER>
```

### Step 4 — Check multipath (iSCSI with multipath)

```bash
# Check multipathd status
systemctl status multipathd
multipath -ll

# Check for failed paths
multipath -ll | grep -i "fail\|ghost"

# Check nova config for multipath
grep "use_multipath_for_image_xfer\|volume_use_multipath" /etc/nova/nova.conf
```

### Step 5 — Check driver-specific Cinder logs

```bash
journalctl -u devstack@c-vol --since "10 minutes ago" | grep -i "error\|traceback\|volume"
tail -200 /var/log/cinder/cinder-volume.log | grep -i "error\|traceback\|attach"

# Check which backend is configured
grep -A 20 '\[DEFAULT\]' /etc/cinder/cinder.conf | grep "enabled_backends\|volume_driver"
```

### Common Resolutions

| Cause | Resolution |
|---|---|
| Volume in error state | `openstack volume set --state available <VOL_ID>` then retry |
| iSCSI initiator not configured | Install `open-iscsi`, configure initiator name |
| RBD keyring missing on compute | Copy client.cinder and client.nova keyrings to compute nodes |
| Multipath misconfigured | Fix `/etc/multipath.conf`, restart `multipathd` |
| Cinder volume service down | Restart `cinder-volume`, check connectivity to storage backend |
| Stuck in 'attaching' state | `openstack volume set --state available <VOL_ID>` and reset attachment |

---

## Slow API Responses

**Symptom**: API calls take several seconds or time out. Performance has degraded from baseline.

### Step 1 — Check database query performance

```bash
# Check for slow queries and active connections
mysqladmin -u root -p processlist
mysql -u root -p -e "SHOW FULL PROCESSLIST;"

# Check slow query log
mysql -u root -p -e "SHOW VARIABLES LIKE 'slow_query_log%';"
mysql -u root -p -e "SHOW VARIABLES LIKE 'long_query_time';"

# Check for table locks
mysql -u root -p -e "SHOW OPEN TABLES WHERE In_use > 0;"

# Check InnoDB status for deadlocks
mysql -u root -p -e "SHOW ENGINE INNODB STATUS\G" | grep -A 30 "LATEST DETECTED DEADLOCK"
```

Enable the slow query log if not already on:

```sql
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = 1;
SET GLOBAL slow_query_log_file = '/var/log/mysql/slow.log';
```

### Step 2 — Check RabbitMQ queue depth

```bash
# List all queues with depth and consumer count
rabbitmqctl list_queues name messages consumers message_bytes

# Filter for deep queues
rabbitmqctl list_queues name messages consumers | awk '$2 > 100'

# Check if workers are consuming
rabbitmqctl list_queues name messages consumers | grep "^nova\." | sort -k2 -rn | head -20

# Check connections
rabbitmqctl list_connections name state | head -30

# Memory and disk alarms
rabbitmqctl status | grep -E "memory|disk|alarm"
```

Queue depth growing without consumers means the workers for that service are not running or are crashing.

### Step 3 — Check service worker count

```bash
# Nova API workers
grep "api_workers\|osapi_compute_workers\|metadata_workers" /etc/nova/nova.conf

# Neutron API workers
grep "api_workers" /etc/neutron/neutron.conf

# Check actual running processes
ps aux | grep nova-api | grep -v grep | wc -l
ps aux | grep neutron-server | grep -v grep | wc -l

# Check worker memory usage
ps aux | grep nova-api | sort -k6 -rn | head -5
```

Increase workers if fewer than `(CPU_COUNT / 2)`:

```ini
# /etc/nova/nova.conf
[DEFAULT]
osapi_compute_workers = 8
metadata_workers = 4
```

### Step 4 — Check Keystone token validation cache

Every API call validates a token against Keystone. Poor caching multiplies the load:

```bash
# Check if caching is enabled
grep -A 10 '\[cache\]' /etc/nova/nova.conf
grep -A 10 '\[keystone_authtoken\]' /etc/nova/nova.conf | grep "memcache\|cache"

# Check memcached
systemctl status memcached
echo "stats" | nc localhost 11211 | grep -E "curr_connections|cmd_get|get_hits|get_misses"

# Hit rate (should be > 80%)
echo "stats" | nc localhost 11211 | awk '/get_hits/{h=$2} /get_misses/{m=$2} END{print "hit_rate:", h/(h+m)*100"%"}'
```

Enable memcached-backed token cache:

```ini
# /etc/<service>/<service>.conf
[keystone_authtoken]
memcached_servers = localhost:11211
token_cache_time = 300
```

### Step 5 — Check service and OS metrics

```bash
# CPU saturation
top -b -n1 | head -20
vmstat 1 5

# Disk I/O
iostat -x 1 5

# Network
netstat -s | grep -i "retransmit\|error"
ss -s
```

### Common Resolutions

| Cause | Resolution |
|---|---|
| Slow DB queries | Add indexes, tune `innodb_buffer_pool_size`, kill long-running queries |
| RabbitMQ queue backlog | Scale up service workers, investigate crashing consumers |
| Too few API workers | Increase `api_workers` in service config, restart |
| No token cache | Enable memcached-backed token cache |
| Disk I/O bottleneck | Move DB to SSD, tune `innodb_flush_method` |
| Network congestion | Check MTU, check for packet loss between nodes |

---

## Service Won't Start

**Symptom**: A service fails to start or crashes immediately after starting.

### Step 1 — Check configuration syntax

```bash
# Nova
nova-manage config validate

# Cinder
cinder-manage config validate

# For services without config validate — try a dry run
nova-api --config-file /etc/nova/nova.conf --help > /dev/null 2>&1; echo $?

# Check for unknown config options (oslo.config)
grep -i "unknown option\|unrecognized section" /var/log/<service>/<service>.log | head -20
```

### Step 2 — Check database connectivity

```bash
# Verify credentials in service config
grep "^connection" /etc/<service>/<service>.conf 2>/dev/null || \
  grep "connection" /etc/<service>/<service>.conf | grep -v "^#"

# Test connection
mysql -u <USER> -p<PASS> -h <HOST> <DB> -e "SELECT 1"

# Check MySQL is running and accessible
systemctl status mysql
systemctl status mariadb
ss -tlnp | grep 3306
```

### Step 3 — Check database migration version

```bash
nova-manage db version
nova-manage api_db version
neutron-db-manage current
cinder-manage db version
glance-manage db_version
keystone-manage db_version
```

If the DB schema is ahead of the code (downgrade scenario) or behind (migration not run), the service will refuse to start.

```bash
# Run pending migrations (always backup DB first)
mysqldump -u root -p <SERVICE_DB> > /tmp/<service>-db-backup-$(date +%Y%m%d).sql
nova-manage db sync
nova-manage api_db sync
neutron-db-manage upgrade heads
cinder-manage db sync
glance-manage db_sync
```

### Step 4 — Check log file permissions

```bash
# Check log directory ownership
ls -la /var/log/<service>/

# Fix ownership
chown -R <service_user>:<service_group> /var/log/<service>/

# Check /run directory for PID files
ls -la /var/run/<service>/ 2>/dev/null || ls -la /run/<service>/ 2>/dev/null
chown -R <service_user>:<service_group> /run/<service>/
```

### Step 5 — Check port conflicts

```bash
# Nova API default: 8774, Neutron: 9696, Cinder: 8776, Glance: 9292, Keystone: 5000
ss -tlnp | grep "8774\|9696\|8776\|9292\|5000"

# Find what is using a port
ss -tlnp sport = :8774
fuser 8774/tcp
```

### Step 6 — Read the service log directly

```bash
# Start the service manually in foreground for immediate output
sudo -u nova nova-api --config-file /etc/nova/nova.conf 2>&1 | head -50
sudo -u neutron neutron-server --config-file /etc/neutron/neutron.conf 2>&1 | head -50

# Or read the log right after attempted start
journalctl -u nova-api -n 100 --no-pager
```

### Common Resolutions

| Cause | Resolution |
|---|---|
| Config syntax error | Fix the offending option (check log for the option name) |
| DB connection refused | Start MySQL/MariaDB, fix connection string |
| DB migration mismatch | Run `<service>-manage db sync` |
| Port already in use | Kill conflicting process or change service port |
| Log dir permissions | `chown -R <user>:<group> /var/log/<service>/` |
| Missing Python package | Install missing dependency (`pip install <package>`) |

---

## RabbitMQ Issues

**Symptom**: Services report connection errors, timeouts, or queue-related failures. `journalctl` shows `AMQP connection error` or `ConnectionClosed`.

### Step 1 — Check RabbitMQ status

```bash
rabbitmqctl status
# Look for: running applications, Erlang version, memory, disk, alarms

rabbitmqctl cluster_status
# Check all nodes are running, no network partitions
# Look for: [{running_nodes,...}] and [{partitions,[]}]  -- partitions must be empty
```

### Step 2 — Check memory and disk alarms

```bash
rabbitmqctl status | grep -A 10 "memory\|disk\|alarm"

# Memory alarm blocks all publishers when vm_memory_high_watermark is exceeded
# Disk alarm blocks all publishers when free disk < disk_free_limit

# Current usage
rabbitmqctl status | grep "memory,"
df -h /var/lib/rabbitmq
```

Resolve memory alarm:

```bash
# Temporarily raise the watermark (edit /etc/rabbitmq/rabbitmq.conf for permanent)
rabbitmqctl set_vm_memory_high_watermark 0.6

# Or purge old messages from dead queues
rabbitmqctl list_queues name messages | awk '$2 > 10000 {print $1}' | \
  xargs -I{} rabbitmqctl purge_queue {}
```

### Step 3 — Check queue depth and consumers

```bash
# Full queue listing
rabbitmqctl list_queues name messages consumers durable auto_delete

# Queues with messages and no consumers (orphaned)
rabbitmqctl list_queues name messages consumers | awk '$3 == 0 && $2 > 0'

# Connection count per service
rabbitmqctl list_connections client_properties state | head -40
```

### Step 4 — Check for network partitions

```bash
rabbitmqctl cluster_status | grep partition
# Output must show: {partitions,[]}
# If it shows nodes, there is an active partition
```

Recover from a split-brain partition (choose the side that has the most current data):

```bash
# Stop RabbitMQ on the minority side
rabbitmqctl stop_app
# Reset it
rabbitmqctl reset
# Re-join the cluster
rabbitmqctl join_cluster rabbit@<MAJORITY_NODE>
# Start
rabbitmqctl start_app
```

### Step 5 — Check OpenStack service connection config

```bash
grep "transport_url" /etc/nova/nova.conf
grep "transport_url" /etc/neutron/neutron.conf
grep "transport_url" /etc/cinder/cinder.conf
# Format: rabbit://<user>:<password>@<host>:5672/<vhost>

# Test connectivity from service host
nc -zv <RABBITMQ_HOST> 5672

# Verify the OpenStack vhost exists
rabbitmqctl list_vhosts
rabbitmqctl list_permissions -p /openstack
```

### Common Resolutions

| Cause | Resolution |
|---|---|
| Memory alarm | Reduce memory use, purge dead queues, increase watermark |
| Disk alarm | Free disk space on RabbitMQ node, purge old messages |
| Network partition | Reset minority node, re-join cluster |
| Wrong vhost/credentials | Fix `transport_url` in service config |
| Too many connections | Tune `vm_memory_high_watermark`, add connection pooling |
| Erlang cookie mismatch | Ensure all cluster nodes share the same `.erlang.cookie` |

---

## Database Issues

**Symptom**: Services log `OperationalError`, `Lost connection to MySQL`, connection pool errors, or very slow queries.

### Step 1 — Check connection pool and active connections

```bash
# Show all active connections
mysql -u root -p -e "SHOW PROCESSLIST;"
mysql -u root -p -e "SHOW FULL PROCESSLIST;"

# Count connections by user/DB
mysql -u root -p -e "
  SELECT user, db, COUNT(*) AS connections, state
  FROM information_schema.processlist
  GROUP BY user, db, state
  ORDER BY connections DESC;"

# Current and max connections
mysql -u root -p -e "SHOW STATUS LIKE 'Threads_connected';"
mysql -u root -p -e "SHOW VARIABLES LIKE 'max_connections';"

# Check connection errors
mysql -u root -p -e "SHOW STATUS LIKE 'Connection_errors%';"
```

### Step 2 — Identify slow queries

```bash
# Active slow queries
mysql -u root -p -e "SHOW FULL PROCESSLIST;" | awk '$6 > 5'  # queries running > 5s

# Kill a long-running query
mysql -u root -p -e "KILL QUERY <PROCESS_ID>;"
mysql -u root -p -e "KILL <PROCESS_ID>;"  # kill the connection

# Enable slow query log temporarily
mysql -u root -p -e "SET GLOBAL slow_query_log = 'ON';"
mysql -u root -p -e "SET GLOBAL long_query_time = 2;"
tail -f /var/log/mysql/mysql-slow.log
```

### Step 3 — Check for table locks

```bash
mysql -u root -p -e "SHOW OPEN TABLES WHERE In_use > 0;"
mysql -u root -p -e "SHOW ENGINE INNODB STATUS\G" | grep -A 50 "TRANSACTIONS"
mysql -u root -p -e "SELECT * FROM information_schema.INNODB_TRX\G"
```

### Step 4 — Check replication lag (HA deployments)

```bash
# On the replica
mysql -u root -p -e "SHOW SLAVE STATUS\G"
# Check: Seconds_Behind_Master, Slave_IO_Running, Slave_SQL_Running
# Slave_IO_Running and Slave_SQL_Running must both be 'Yes'
# Seconds_Behind_Master should be < 30

# Galera cluster status
mysql -u root -p -e "SHOW STATUS LIKE 'wsrep%';"
mysql -u root -p -e "SHOW STATUS LIKE 'wsrep_cluster_size';"
mysql -u root -p -e "SHOW STATUS LIKE 'wsrep_local_state_comment';"
# wsrep_local_state_comment should be 'Synced'
```

### Step 5 — Check InnoDB buffer pool usage

```bash
mysql -u root -p -e "SHOW STATUS LIKE 'Innodb_buffer_pool%';"
# Innodb_buffer_pool_reads / Innodb_buffer_pool_read_requests gives disk read ratio
# Should be < 1%

# Current buffer pool size
mysql -u root -p -e "SHOW VARIABLES LIKE 'innodb_buffer_pool_size';"
# Typically set to 70-80% of available RAM on dedicated DB servers
```

### Step 6 — Check service-side connection pool config

```bash
# Nova DB pool settings
grep -E "max_pool_size|max_overflow|pool_timeout|connection_recycle_time" /etc/nova/nova.conf

# Typical tuning
# [database]
# max_pool_size = 20
# max_overflow = 10
# pool_timeout = 30
# connection_recycle_time = 3600
```

### Common Resolutions

| Cause | Resolution |
|---|---|
| Too many connections | Increase `max_connections` in my.cnf, tune service pool size |
| Slow queries | Add missing indexes, kill runaway queries, tune buffer pool |
| Table locks / deadlocks | Identify and kill blocking transactions, review application retry logic |
| Galera node desynced | Restart the desynced node, allow SST/IST to resync |
| Replication lag | Reduce write load on primary, check replica I/O |
| Connection pool exhausted | Increase `max_pool_size`, reduce `max_overflow` |
