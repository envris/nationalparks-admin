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


### PostgreSQL RDS point-in-time recovery (PITR):
With enabled automated backup option for RDS instance it’s possible to restore instance anytime in the past (limited by backup retention policy). There are 2 ways to do restoration:

* Using cli (aws command line interface)
    ```
    aws rds restore-db-instance-to-point-in-time \
        --source-db-instance-identifier mysourcedbinstance \
        --target-db-instance-identifier mytargetdbinstance \
        --restore-time 2009-10-14T23:45:00.000Z  
    ```

    ```--use-latest-restorable-timecan``` can be specified

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
