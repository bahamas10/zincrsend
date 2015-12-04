zincrsend - ZFS Incremental Send
================================

Incremental ZFS send/recv backup script

Configuration
-------------

As of right now this script does not take any arguments or environmental variables,
all configuration must be done in the script itself.

Note: this script *should* take arguments or a config file - I was just lazy

At the top of the script you'll see a `config` section... modify it
to fit your environment

``` bash
#########
# config
#########

# datasets to send
datasets=(
)
# datasets to send recursively
# even if a dataset does not have any descendant datasets, using a
# recursive zfs send (-R) is preferable because it will result
# in locally removed snapshots being removed on the remote end as well
datasets_recursive=(
	goliath/public
	goliath/minecraft
	goliath/backups
)

# information about the server on the receiving end
remote_server='my-backup-server.example.com'
remote_user='dave'
remote_port='22'
remote_dataset='paper' # zpool name most likely
remote_command_prefix='sudo' # leave blank for nothing
remote_ssh_opts=(-i /root/backup/backup.key) # additional opts to give to ssh

# prefix to use for snapshots created by this script
snapshot_prefix='zincrsend_'

# higher = more output
verbosity_level=0

# function to execute at the end - can be anything
# $1 - exit code - the code that will be used when this script exits
# returns - nothing
end() {
	local exitcode=$1
	local msg=
	case "$exitcode" in
		0) msg='ok';;
		*) msg='failed';;
	esac
	/opt/custom/bin/pushover zincrsend "$msg - took $((SECONDS / 60)) minutes"
}

#############
# end config
#############
```

Example
-------

```
# ./zincrsend
starting on Fri Dec  4 11:16:00 UTC 2015

processing dataset: goliath/public

creating snapshot locally: goliath/public@zincrsend_1449227760
latest remote snapshot: paper/public@zincrsend_1449173284
zfs sending (incremental) @zincrsend_1449173284 -> goliath/public@zincrsend_1449227760 to paper/public
receiving incremental stream of goliath/public@zincrsend_1449227760 into paper/public@zincrsend_1449227760
received 312B stream in 1 seconds (312B/sec)

processing dataset: goliath/minecraft

creating snapshot locally: goliath/minecraft@zincrsend_1449227763
latest remote snapshot: paper/minecraft@zincrsend_1449173286
zfs sending (incremental) @zincrsend_1449173286 -> goliath/minecraft@zincrsend_1449227763 to paper/minecraft
receiving incremental stream of goliath/minecraft@zincrsend_1449227763 into paper/minecraft@zincrsend_1449227763
received 312B stream in 1 seconds (312B/sec)

processing dataset: goliath/backups

creating snapshot locally: goliath/backups@zincrsend_1449227766
latest remote snapshot: paper/backups@zincrsend_1449173288
zfs sending (incremental) @zincrsend_1449173288 -> goliath/backups@zincrsend_1449227766 to paper/backups
receiving incremental stream of goliath/backups@zincrsend_1449227766 into paper/backups@zincrsend_1449227766
received 312B stream in 1 seconds (312B/sec)
receiving incremental stream of goliath/backups/dave@zincrsend_1449227766 into paper/backups/dave@zincrsend_1449227766
received 217MB stream in 367 seconds (607KB/sec)
receiving incremental stream of goliath/backups/skye@zincrsend_1449227766 into paper/backups/skye@zincrsend_1449227766
received 312B stream in 1 seconds (312B/sec)
receiving incremental stream of goliath/backups/theinsurancemarkets@zincrsend_1449227766 into paper/backups/theinsurancemarkets@zincrsend_1449227766
received 312B stream in 1 seconds (312B/sec)
receiving incremental stream of goliath/backups/dad@zincrsend_1449227766 into paper/backups/dad@zincrsend_1449227766
received 312B stream in 1 seconds (312B/sec)
receiving incremental stream of goliath/backups/web@zincrsend_1449227766 into paper/backups/web@zincrsend_1449227766
received 3.16MB stream in 5 seconds (647KB/sec)

script ran for ~6 minutes (384 seconds)

pushover sent!

---------------------------------
```

Notes
-----

Consider using [ZFS Prune Snapshots](https://github.com/bahamas10/zfs-prune-snapshots) to remove
old `zincrsend` snapshots

License
-------

MIT License
