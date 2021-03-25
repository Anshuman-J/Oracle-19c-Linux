# Oracle Database 19c QSG for EXPRESSCLUSTER X (Linux data mirror)

## About This Guide

This guide helps you to integrate Oracle Database 19c with 2 mirror nodes using the EXPRESSCLUSTER X (ECX).The guide is only for those who have basic understanding,knowledge and setup skills of EXPRESSCLUSTER X.

For the detailed information of EXPRESSCLUSTER X, please refer to [this site](https://www.nec.com/en/global/prod/expresscluster/index.html) .


## Configurations Description
In this document, create 2 nodes (Node1 and Node2 as below) mirror disk type cluster.
Prepare Oracle HA By using EXPRESSCLUSTER X. 


## Software Versions
- Oracle 19c    
- EXPRESSCLUSTER X 4.2 for Linux (internal version：4.2.0-1)
- EXPRESSCLUSTER X license
  - EXPRESSCLUSTER X 4.2 for Linux
  - EXPRESSCLUSTER X Replicator 4.2 for Linux

## Cluster Configurations
- Group resources
  - exec resource
  - floting IP resource
  - mirror disk resource
- Monitor rerources
  - floting IP monitor resource
  - mirror disk connect monitor resource
  - mirror disk monitor resource
  - exec monitor resource
  
## Oracle Prerequisites and Installation Procedure
  - System Requirements for oracle-19c
  - Please refer and check to [this site](https://oracle-base.com/articles/19c/oracle-db-19c-installation-on-oracle-linux-7) 
 
 OR
  - Please refer [this site](https://www.server-world.info/en/note?os=CentOS_8&p=oracle19c&f=1)

## Pre-checks before Installation :-

  - Hostname should configure on the both servers.
  - Also, set the fully qualified hostname is configured into /etc/hostname and /etc/hosts.
  - To completely disable SELinux on CentOS, open `/etc/selinux/config` file with a text editor and set the line SELINUX to disabled.
  - Firewall should be in closed state.
  - Make sure before installion a swap is created on the both machine.   

## EXPRESSCLUSTER setup  
  - Let us consider the following 2 node cluster and try to understand it.

  Cluster Information:-
  ---
  ||Node1(Active)|Node2(Stanby)|
  |---|---|---|
  |Server Name|Server1|Server2|
  |IPaddress|10.0.7.128|10.0.7.129|  
  |cluster partition|/dev/sdb1|/dev/sdb1|
  |data partition|/dev/sdc2|/dev/sdc2|
    
  Failover Group Information:-
  ---
  |Parameter|Value|
  |---|---|
  |Name|Failover1|
  |Startup Server| Server1 -> Server2 |
  |Floating ip address|10.0.7.130|
  |Mirror disk resource (mount point))|/oradata/ORCL|

  - In Config mode of the Cluster WebUI, add failover group to use Db_oracle.  
  - You need to add the following resources.
      - Floating ip resource  
      - Mirror disk resource  
  - If you want to know how to add resource, please refer to [this site](https://github.com/EXPRESSCLUSTER/BasicCluster/blob/master/X41/Lin/2nodesMirror_Lin.md#how-to-setup-basic-2-nodes-mirror-cluster-on-linux) 
                                                          
 OR
  - please refer to how to setup basic 2 nodes mirror cluster on Linux [this site](https://github.com/EXPRESSCLUSTER/BasicCluster/blob/master/X41/Lin/2nodesMirror_Lin.md)


### Database configuration

- SID name               : orcl
- Database name          : ORCL
- Oracle home            : `/home/oracle`
- Database files location: `/oradata/ORCL`


## Moving Oracle Database File to the Mirror Data Partition

### Changing the location of the Database files (*.DBF) on the Primary Server

- All the default  (*.DBF) file path is /u01/app/oracle/oradata/ORCL2

- Switch to oracle user and export the ORACLE_SID environment      variable to target SID (example: orcl):
           
         export ORACLE_SID= orcl

- Start SQLPlus in command console:
         
         sqlplus as sysdba

- Connect to target database instance:
    
         Enter user-name: /as sysdba

- You should see the status at the SQLPlus prompt as:

    --> Connected

- Stop target database instance:
         
        SQL> shutdown immediate;

- Check the status at the SQL prompt as below:
   - Database closed.
   - Database dismounted.
   - ORACLE instance shut down.

 - Copy the database (*.DBF) files from default directory to    mirrored disk directory Some typical database files include the following:
  
   - SYSAUX01.DBF
   - SYSTEM01.DBF
   - UNDOTBS01.DBF
   - USERS01.DBF
           
          1) scp -r  /u01/app/oracle/oradata/ORCL/sysaux01.dbf    /oradata/ORCL/

          2) scp -r  /u01/app/oracle/oradata/ORCL/system01.dbf    /oradata/ORCL/

          3) scp -r  /u01/app/oracle/oradata/ORCL/users01.dbf    /oradata/ORCL/

          4) scp -r  /u01/app/oracle/oradata/ORCL/undotbs01.dbf    /oradata/ORCL/



 ### Note:
     Do not copy the file TEMP01.DBF at this time.

- Restart and mount target database instance:
       
       SQL> startup mount

- Set new location of the database (*.DBF) files by executing the following command for all moved database files:

## SYNTAX 
SQL> alter database move datafile
 'original file path' to 'new file path';

Example: 

     SQL> alter database move datafile '/u01/app/oracle/oradata/ORCL/users01.dbf' to '/oradata/ORCL/users01.dbf';

  -->  Database altered.

     SQL> alter database move datafile '/u01/app/oracle/oradata/ORCL/sysaux01.dbf' to '/oradata/ORCL/sysaux01.dbf';

  --> Database altered.

     SQL> alter database move datafile '/u01/app/oracle/oradata/ORCL/system01.dbf' to '/oradata/ORCL/system01.dbf';

  --> Database altered.

     SQL> alter database move datafile '/u01/app/oracle/oradata/ORCL/undotbs01.dbf' to '/oradata/ORCL/undotbs01.dbf';

  --> Database altered.

 - Open target database instance:
      
        SQL> alter database open;

You should see the status at the SQL prompt as:

Database altered.

- Verify new database file locations:
    
       SQL> select name from v$datafile;

  - The command output should show new location of moved database files.


- Create new temporary database file by executing the following command at the prompt:


        SQL> create temporary tablespace temp2 tempfile '/oradata/ORCL/temp02.dbf' size 50M extent management local;

   --> Tablespace created.

  - Adjust the command options as per need.

- See the status at the SQL prompt as:

  --> Tablespace created.

- Configure new default temporary database file by executing the following command at the prompt:

       SQL> alter database default temporary tablespace temp2;

   - See the status at the SQL prompt as:

     --> Database altered.


- Remove original temporary tablespace and data files by executing the following command at the prompt: 

       SQL> drop tablespace temp including contents and datafiles;

    - You should see the status at the SQL prompt as:

      --> Tablespace altered.

### This completes the re-configuration of the database (*.DBF) files to the mirror partition \oradata.


## Changing the location of the log files (*.log) on the Primary Server 

- Shutdown target database instance:
     
        SQL> shutdown immediate;

- Copy log (*.LOG) files from default directory (example: /u01/app/oracle/oradata/orcl) to mirrored disk directory (example: /oradata/orcl). 

 - Some typical database log files include the following:
   - REDO01.LOG
   - REDO02.LOG
   - REDO03.LOG        
   
EXAMPLE :-

         mv /u01/app/oracle/oradata/ORCL/redo01.log    /oradata/ORCL/

         chown oracle:oinstall redo01.log
      
         mv /u01/app/oracle/oradata/ORCL/redo02.log    /oradata/ORCL/
       
         chown oracle:oinstall redo02.log

         mv /u01/app/oracle/oradata/ORCL/redo01.log    /oradata/ORCL/
      
         chown oracle:oinstall redo03.log

 - Restart and mount target database instance:
       
        SQL> startup mount        

  - Set new location of the log files by executing the following command pattern for all moved log files:

  EXAMPLE:- 

        SQL>  alter database rename file '/u01/app/oracle/oradata/ORCL/redo01.log' to '/oradata/ORCL/redo01.log';

        SQL>  alter database rename file '/u01/app/oracle/oradata/ORCL/redo02.log' to '/oradata/ORCL/redo02.log';  

        SQL>  alter database rename file '/u01/app/oracle/oradata/ORCL/redo03.log' to '/oradata/ORCL/redo03.log';    


-  Reopen target database instance:
        
         SQL> alter database open;

-  Check log file location 

       SQL>  select * from v$logfile;       

 #### Completed the re-configuration of the log files to the mirror partition \oradata.      

 ## Changing the location of the Control files (*.ctl) on the Primary Server

 - Shutdown target database instance:
     
     SQL> shutdown immediate;

 -  Modify target Oracle instance SPFILE

 -  Execute the following SQLPlus command pattern to create PFILE for modification
         
         SQL> create pfile from spfile;

- see the status at the SQL prompt similar to:

     --> File created.

 - Switch to rrot user and Copy control (*.CTL) files from default directories to Oracle instance data directory on the mirrored disk (example: /oradata/orcl). Typically, the database control files include the following:

     - CONTROL01.CTL
     - CONTROL02.CTL  

     EXAMPLE :-

            [root@db1 ORCL]# scp -r  /u01/app/oracle/oradata/ORCL/control*    /oradata/ORCL/

            [root@db1 ORCL]# chown oracle:oinstall control01.ctl

            [root@db1 ORCL]# chown oracle:oinstall control02.ctl

- After 
                      
      [root@db1 ~]# find /home -name initorcl.ora
/home/oracle/dbs/initorcl.ora


          [oracle@db1 dbs]$ vi initorcl.ora
          ----------------------------------------------------
          *.control_files='/oradata/ORCL/control01.ctl','/oradata/ORCL/control02.ctl'
          ----------------------------------------------------
              
   -  Edit the newly created PFILE and change directory paths in for the parameter “control_files” to the mirrored disk path.

- Restart target database instance with new parameter file:

       SQL> startup

- Verify control file location changes:
 
        SQL> show parameter control_files;

All the below commands should be run by the oracle user 

 /etc/oratab

change like follows

db01:/usr/oracle/database:Y

vi .bash_profile 


export ORACLE_HOME=/home/oracle

export ORACLE_SID=orcl

sqlplus /nolog

conn / as sysdba

