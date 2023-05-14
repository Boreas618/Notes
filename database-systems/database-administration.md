# **Database Administration**

## **Capacity Planning**

When implementing a database, it is important to consider two things:

- disk space requirements
- transaction throughput

These factors need to be considered both at the time of implementation and throughout the life of the system.

Table size is the sum of all table sizes. Table size = \# rows \times width

## **Backup and Recovery**

### **Physical vs. Logical Backup**

**Physical backup**

- Raw copies of files and directories.
- Suitable for large databases that need fast recovery.
- Database is preferably offline ("cold backup") when backup occurs.
  - MySQL Enterprise automatically handles file locking, so database is not wholly offline.
- Backup = exact copies of the database directories and files.
- Backup should include logs.
- Backup is only portable to machines with a similar configuration to restore.

Procedure:

- Shut down DBMS.
- Copy backup over current structure on disk.
- Restart DBMS.

**Logical backup**

- Backup completed through SQL queries.
- Slower than physical.
  - SQL Selects rather than OS copy.
- Output is larger than physical.
- Doesn't include log or config files.
- Machine independent.
- Server is available during the backup.
- In MySQL, you can use the backup using `mysqldump` or `SELECT INTO OUTFILE` to restore.
- Use `mysqlimport`, or `LOAD DATA INFILE` within the mysql client.

### **Online vs. Offline Backup**

**Online (or HOT) backup**

- Backups occur when the database is “live”.
- Clients don't realize a backup is in progress.
- Need to have appropriate locking to ensure integrity of data.

**Offline (or COLD) backup**

- Backups occur when the database is stopped.
- To maximize availability to users, take backup from replication server not live server.
- Simpler to perform cold backup is preferable, but not available in all situations e.g. applications without downtime.

### **Full vs. Incremental Backup**

**Full**

- A full backup is where the complete database is backed up.
  - May be Physical or Logical, Online or Offline.
- It includes everything you need to get the database operational in the event of a failure.

**Incremental**

- Only the changes since the last backup are backed up.
- For most databases, this means only backup log files.
- To restore:
  - Stop the database, copy backed up log files to disk.
  - Start the database and tell it to redo the log files.

### **Onsite vs. Offsite Backup**

In the same device?
