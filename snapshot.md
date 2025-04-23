Make directory

```
mkdir /mnt/snapshot
```
Permissions

```
chown wazuh-indexer:wazuh-indexer /mnt/snapshots
```
Edit Wazuh indexer YAML file

```
vi /etc/wazuh-indexer/opensearch.yml
```
 Add the following
 
```
path.repo: ["/mnt/snapshots"]
```
Save and close file.

Then register the repository using the REST API:
Esure the dev tools used is under Index managment.

```
PUT /_snapshot/my-fs-repository
{
  "type": "fs",
  "settings": {
    "location": "/mnt/snapshots"
  }
}
```

### Create policy

Make a policy name.
Choose the indices you wish to backup.
Snapshot schedule will be on a daily frequency.
Create a Snapshot retention period.
Notification configurations.



 



