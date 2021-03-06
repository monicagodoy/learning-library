# Query Your Data

## Introduction

*Describe the lab in one or two sentences, for example:* This lab walks you through the steps to ...

Estimated Lab Time: n minutes

### About Product/Technology
Enter background information here..

### Objectives

*List objectives for the lab - if this is the intro lab, list objectives for the workshop*

In this lab, you will:
* Objective 1
* Objective 2
* Objective 3

### Prerequisites

*Use this section to describe any prerequisites, including Oracle Cloud accounts, set up requirements, etc.*

* An Oracle Free Tier, Always Free, Paid or LiveLabs Cloud Account
* Item no 2 with url - [URL Text](https://www.oracle.com).

*This is the "fold" - below items are collapsed by default*

## **STEP 1**: title

In this part of the Hands-On Lab you will now plugin UPGR into CDB2.

We could have done this with AutpUpgrade already – you can see this in the OPTIONAL AutoUpgrade exercise (Parameter: target_cdb=CDB2). But we rather decided that you should do these steps manually to understand the implications.

CDB2 is a Multitenant Container database.
And UPGR will be converted into a PDB, and then become a pluggable database.

The key is, that – in order to plugin a non-CDB such as the UPGR database – it has to be upgraded first to the same release as the CDB it gets plugged into.
Index

    1. Preparation UPGR as non-CDB
    2. Compatibility check
    3. Plugin Operation

1. Preparation UPGR as non-CDB

Switch to the UPGR database in 18c environment:

. upgr19
sqlplus / as sysdba

Shutdown UPGR and start it up read only:

shutdown immediate
startup open read only;

Create the XML manifest file describing UPGR’s layout and information:

exec DBMS_PDB.DESCRIBE('/home/oracle/pdb1.xml');

Shutdown UPGR:

shutdown immediate
exit

Switch to CDB2:

. cdb2
sqlplus / as sysdba
2. Compatibility check

Ideally you do a compatibility check before you plugin finding out about potential issues. This step is not mandatory but recommended. The check will give you YES or NO.

Compatibility check:

set serveroutput on

DECLARE
compatible CONSTANT VARCHAR2(3) := CASE DBMS_PDB.CHECK_PLUG_COMPATIBILITY( pdb_descr_file => '/home/oracle/pdb1.xml', pdb_name => 'PDB1') WHEN TRUE THEN 'YES' ELSE 'NO'
END;
BEGIN
DBMS_OUTPUT.PUT_LINE('Is the future PDB compatible? ==> ' || compatible);
END;
/

If the result is “NO” (and it is NO very often), then don’t be in panic.
Check for TYPE='ERROR' in PDB_PLUG_IN_VIOLATIONS.

In this case, the result should be “YES“.
3. Plugin Operation

Plugin UPGR with its new name PDB1 – from this point there’s no UPGR database anymore. In a real world environment, you would have a backup or use a backup/copy to plug in. In our lab the database UPGR will stay in place and become PDB1 as part of CDB2.

Please use the proposed naming as the FILE_NAME_CONVERT parameter and TNS setup have been done already.
Use the NOCOPY option for this lab to avoid additional copy time and disk space consumption. The show pdbs command will display you all existing PDBs in this CDB2.

create pluggable database PDB1 using '/home/oracle/pdb1.xml' nocopy tempfile reuse;
show pdbs

As you couldn’t do a compatibility check beforehand, you’ll open the PDB now and you will recognize that it opens only with errors.

alter pluggable database PDB1 open;

Find out what the issue is:

column message format a50
column status format a9
column type format a9
column con_id format 9

select con_id, type, message, status from PDB_PLUG_IN_VIOLATIONS
where status<>'RESOLVED' order by time;

As you can see, a lot of the reported issues aren’t really issues. This is a known issue. Only in the case you see ERROR in the first column you need to solve it.

The only real ERROR says:

PDB plugged in is a non-CDB, requires noncdb_to_pdb.sql be run.

Kick off this sanity script to adjust UPGR and make it a “real” pluggable database PDB1 with noncdb_to_pdb.sql. Runtime will vary between 10-20 minutes. Take a break while it is running. The forced recompilation takes quite a bit.

alter session set container=PDB1;
@?/rdbms/admin/noncdb_to_pdb.sql

Now SAVE STATE. This ensures, that PDB1 will be opened automatically whenever you restart CDB2. Before you must restart the PDB as otherwise it opens only in RESTRICTED mode.

shutdown
startup
alter pluggable database PDB1 save state;
alter session set container=CDB$ROOT;
show pdbs
exit

Try to connect directly to PDB1 – notice that you can’t just connect without specifying the service name as PDB1 is not visible on the OS level.

sqlplus "sys/oracle@pdb1 as sysdba"

exit

As alternative you could also use the EZconnect (speak: Easy Connect)

sqlplus "sys/oracle@//localhost:1521/pdb1 as sysdba"

exit

You may now [proceed to the next lab](#next).

## Learn More

*(optional - include links to docs, white papers, blogs, etc)*

* [URL text 1](http://docs.oracle.com)
* [URL text 2](http://docs.oracle.com)

## Acknowledgements
* **Author** - <Name, Title, Group>
* **Contributors** -  <Name, Group> -- optional
* **Last Updated By/Date** - <Name, Group, Month Year>
* **Workshop (or Lab) Expiry Date** - <Month Year> -- optional, use this when you are using a Pre-Authorized Request (PAR) URL to an object in Oracle Object Store.

## Need Help?
Please submit feedback or ask for help using our [LiveLabs Support Forum](https://community.oracle.com/tech/developers/categories/livelabsdiscussions). Please click the **Log In** button and login using your Oracle Account. Click the **Ask A Question** button to the left to start a *New Discussion* or *Ask a Question*.  Please include your workshop name and lab name.  You can also include screenshots and attach files.  Engage directly with the author of the workshop.

If you do not have an Oracle Account, click [here](https://profile.oracle.com/myprofile/account/create-account.jspx) to create one.
