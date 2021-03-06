## Retention Policy

Setting retention policy options allows to remove obsolete backups. Those settings will then be a trade-off between saving space and data retention.

### Full backup retention

Full backup retention is determined by the `repo1-retention-full-type` and `repo1-retention-full` options.

The `repo1-retention-full-type` option can be set to `count` (default) to retain a specific number of full backups. As an example, if `repo1-retention-full-type=count` and `repo-retention-full=2`, _pgBackRest_ will retain the 2 last successful full backups.

Alternatively the `repo1-retention-full-type` option can be set to `time` to retain full backups for a specific time duration. With this configuration, at least one full backup older than the retention period will remain to fulfill the retention requirement. All other full backups older than the `repo1-retention-full` value, measured in days, will be removed from the repository. As an example, if `repo1-retention-full-type=time` and `repo-retention-full=30` (days), one full backup older than 30 days will be preserved. If in this example the repository would hold five full backups - one 25 days old, one 35 days old, and the rest older - the 35-days-old full backup would still be required to restore to the period between 30 and 25 days ago, and as such is kept even though it is older than the configured retention period. All other older backups will be removed during retention maintenance.

When a full backup expires, all differential and incremental backups associated with it will also expire. When the `repo1-retention-full` option is not defined, a warning will be issued.

Archived WAL files that are older than the remaining oldest full backup will be automatically removed, but this behavior can be changed with the archive retention settings (`repo1-retention-archive-type` and `repo1-retention-archive`).

Set the `repo1-retention-full` option to its maximum value (9999999) to retain all backups indefinitely.

### Differential backup retention

Differential backup retention is determined by the `repo1-retention-diff` option.

When a differential backup expires, all incremental backups associated with the differential backup will also expire.

When not defined, all differential backups will be retained until the dependent full backups expires.

### Archive retention

_pgBackRest_ allows to define a specific retention policy for archives.

> :warning: _pgBackRest_ relies on backup start time to define the archives to clean. In some cases, the oldest archived WAL file in the repository might be older than the first backup WAL start location, in which case it will not be removed until the first backup expires.

#### `repo1-retention-archive-type`

The `repo1-retention-archive-type` option can be set to define the backup types counted for WAL retention.

If the `repo1-retention-archive-type` option is set to `full` (default), _pgBackRest_ will keep archived WAL files for the number of full backups as defined by `repo1-retention-archive`.

If the `repo1-retention-archive-type` option is set to `diff` (differential), _pgBackRest_ will keep archives for the number of full and differential backups defined by `repo1-retention-archive`. As an example, if the last backup taken was a full backup, it will be counted as a differential for the purpose of the repository retention.

If the `repo1-retention-archive-type` option is set to `incr` (incremental), _pgBackRest_ will keep archives for the number of full, differential, and incremental backups defined by `repo1-retention-archive`.

> :warning: It is recommended to keep the `repo1-retention-archive-type` option with its default, in which case archives will be removed along with full backups.

#### `repo1-retention-archive`

The `repo1-retention-archive` option can be set to define the number of backups preserved for continuous WAL retention.

> :warning: WAL segments required to make a backup consistent (based on the full and differential backups retention policies) are always retained until the backup expires regardless of how the `repo1-retention-archive` option is configured.

The `repo1-retention-archive` is highly dependent on other options:

- If `repo1-retention-full-type` is set to `time`, then the `repo1-retention-archive` value will default to removing archived WAL files that are older than the oldest full backup retained after satisfying the `repo1-retention-full` setting.

- When `repo1-retention-archive-type` is set to `incr`, the `repo1-retention-archive` must be set.

- If the `repo1-retention-archive` value is not set, and `repo1-retention-full-type` is set to **count** (default), then the `repo1-retention-archive-type` value defines how archives will expire: when `repo1-retention-archive-type` is set to `full` or `diff`, archives expiry will default to the `repo1-retention-full` or `repo1-retention-diff` respectively. This configuration will ensure that the archived WAL files are only removed for corresponding expired backups.

Using the `repo1-retention-archive` setting in conjunction with `repo1-retention-archive-type` can reclaim disk-space by removing extra WAL segments. This configuration is **not recommended** as it negates the ability to perform point-in-time recovery from the backups with expired WAL.

### Glossary

#### pgBackRest

[`repo-retention-archive-type`](https://pgbackrest.org/configuration.html#section-repository/option-repo-retention-archive-type)
[`repo-retention-archive`](https://pgbackrest.org/configuration.html#section-repository/option-repo-retention-archive)
[`repo-retention-diff`](https://pgbackrest.org/configuration.html#section-repository/option-repo-retention-diff)
[`repo-retention-full-type`](https://pgbackrest.org/configuration.html#section-repository/option-repo-retention-full-type)
[`repo-retention-full`](https://pgbackrest.org/configuration.html#section-repository/option-repo-retention-full)
