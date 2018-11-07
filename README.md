# ansible-role-cephfs-client

This role is used to install cephfs and mount directories from (a) Ceph monitor(s) into the target.

In it's current state this role will set up:
1) Install the software dependencies to use cephfs on the target.
2) Mount the cephfs directories into the target.

## Requirements

- Ceph monitors need to be reachable from the target's network.
- A target kernel that supports the cephfs features you want to use.

## Variables

| Variable                | Type        | Description                 |
|-------------------------|-------------|-----------------------------|
| `ceph_monitor`          | list (str)  | List of ceph monitor FQDNs. |
| `cephfs_client_mapping` | list (dict) | List of mappings for dirs.  |

- Note, for `cephfs_client_mapping`, the dictionaries are structured as follows:

   ```
   {
       "src": "/ceph/location",  <--- The directory on the ceph monitor(s).
       "dst": "/local/location", <--- The directory on target to mount the above.
       "id": "cephx_id",         <--- The id of the user for cephx, e.g. "people".
       "key": "cephx_key"        <--- The cephx key corresponding to the user
   }
   ```
   Example in use:

   ```
   cephfs_client_mapping:
     - {"src": "/people", "dst": "/usr/people", "id": "people", "key": "123abc..."}
   ```

## Usage

1. Set the appropriate variables (listed above).
2. Create a playbook that includes this role. Following is an example.
   ```shell
   - hosts: ceph-clients
     become: true
     # The pre_tasks section will disable gathering of
     # facts to first install python2 if it is not
     # already installed and then gather facts.
     gather_facts: false
     pre_tasks:
     - name: Install python2 for Ansible
       raw: bash -c "test -e /usr/bin/python || (apt -qqy update && apt install -qqy python-minimal)"
       register: output
       changed_when: output.stdout != ""
     - name: Gathering Facts
       setup:
     roles:
       # This is actually where this role
       # (ansible-role-cephfs_client) is being included.
       - role: ansible-role-cephfs_client
   ```