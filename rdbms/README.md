RDBMS
=====

TODO:
 * document existing RDBMS setup.
 * document existing backup and recovery measures.
 * document WAL management as it is.
 * raise tickets WAL management as it should be (if required)
 * document procedure for recovery
 * create scripts for recovery (update procedure as it get's faster/simpler)
 * raise tickets for monitoring improvements

Notes:
 * give tickets a RDBMS tag, light blue
 * be consice, link don't over-synthesize


### FAQ:
_**Q: What Amazon RDS offers to guarantee data safe?_**

**A:** Amazon provides 2 different types to save your data "Automated Backups"(http://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_WorkingWithAutomatedBackups.html) and "DB Snapshots"(http://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_CreateSnapshot.html).


_**Q: What's the difference between "Automated Backup" and "DB Snapshots"?_**

**A:** "DB Snapshot" it's a storage volume snapshot(instant) of your DB instance, backing up the entire DB instance and not just individual databases. DB state can be restored ONLY same time when snapshot was created.
"Automated Backup" it's a  continuous process which allows you to restore database anytime in the past and guarantee data safe up to last transaction. It includes daily snapshot + automated WALs(write ahead logs) archiving.


_**Q: Where does Amazon store backups and snapshots?_**

**A:** They are stored in S3.


_**Q: How quick snapshot can be taken?_**

**A:** Once you run command AWS makes a filesystem snapshot and moves it to S3 (it becomes available once completely move to S3). You can verify snapshot status with "db-snapshot-completed".


_**Q: How quick can we restore backup and snapshot?_**

**A:** Restoring automated backup takes more time. It requires to restore database + apply all WALs(write ahead logs) to reach specified point to recover.
Restoring snapshot is much faster, as it needs to restore only filesystem snapshot.


_**Q: Can we use db snapshot to speed up restore after failed release?_**

**A:** Yes, it's best option.
Create snapshot before release, verify it's completed with db-snapshot-completed, run DB related release scripts.

_**Q: Can we clone instance with data for performance testing purpose?_**

**A:** Yes, you can clone any instance you run.
Just do PITR(point in time recovery) for source instance with latest restorable time option. It'll create new target instance with new name but identical data as source has.




### PostgreSQL RDS point-in-time recovery (PITR):
With enabled automated backup option for RDS instance it’s possible to restore instance anytime in the past (limited by backup retention policy). There are 2 ways to do restoration:

* Using cli (aws command line interface)
    ```
    aws rds restore-db-instance-to-point-in-time \
        --source-db-instance-identifier mysourcedbinstance \
        --target-db-instance-identifier mytargetdbinstance \
        --restore-time 2009-10-14T23:45:00.000Z  
    ```

    ```--use-latest-restorable-time``` can be specified

    [AWS Documentation](http://docs.aws.amazon.com//cli/latest/reference/rds/restore-db-instance-to-point-in-time.html)

* Using AWS management console
```
    1. Sign in to the AWS Management Console and open the Amazon RDS console at https://console.aws.amazon.com/rds/.
    2. In the navigation pane, click DB Instances.
    3. Click Instance Actions, and then click Restore To Point In Time.
    The Restore DB Instance window appears.
    4. Click on the Use Custom Restore Time radio button.
    5. Enter the date and time that you wish to restore to in the Use Custom Restore Time text boxes.
    6. Type the name of the restored DB instance in the DB Instance Identifier text box.
    7. Click the Launch DB Instance button.
```

### PostgreSQL RDS DR solutions
  Amazon is very limited in DR options and provides only Multi-AZ solution - switchover to replica running on different availability zone. Switchover occurs automatically and downtime usually is around 1-2 minutes. If failover occurs no application/DNS changes required, everything will be changed by amazon (endpoint will be changed to new instance)

  Alternatively to guarantee redundancy snapshots can be created ([doc](http://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_CreateSnapshot.html)) and copy ([doc](http://docs.aws.amazon.com//cli/latest/reference/rds/copy-db-snapshot.html)) them to another region. *All data since last run will be lost*

  Full data export can be configured and scheduled to store dumps on S3. *all data since last run will be lost*

  To move data outside of RDS can be configured logical replication based on 3rd party software (https://bucardo.org/wiki/Bucardo). Ideally it’ll require separate server(either amazon EC2 instances or datacenter server or any other service providers). This solution doesn’t guarantee 100% data safe. Some data generated between synchronisations can be lost.
