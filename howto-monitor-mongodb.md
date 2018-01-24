# How to Monitor MongoDB with ECX SSS

--------------------------------------------------------------------------------

This document descrives how to setup MongoDB monitoring system with EXPRESSSCLUSTER X SSS

--------------------------------------------------------------------------------

## This procedure was tested with
- CentOS 6.5 (x86_64)
- EXPRESSCLUSTER X 3.3.0-1 for Linux
- MongoDB 2.6

## MongoDB installation
For installation by yum, create /etc/yum.repos.d/mongodb.repo

    # cat /etc/yum.repos.d/mongodb.repo
    [mongodb]
    name=MongoDB Repository
    baseurl=http://downloads-distro.mongodb.org/repo/redhat/os/x86_64/
    gpgcheck=0
    enabled=1

Then install.

    # yum install mongodb-org

## ECX SSS instalation

	# rpm -ivh expresscls-3.3.0-1.x86_64.rpm
    # clplcnsc -I <LicenseFile> -P XSSS33
    # reboot

Open web browser and access to http://localhost:29003/  
Add **custom monitor** resource for MongoDB. Name it as **genwMontgoDB** and edit genw.sh as below.

	#! /bin/sh
	#***********************************************
	#*                   genw.sh                   *
	#***********************************************
	
	ulimit -s unlimited
	mongo --quiet clpmongow.js
	exit $?

Create  
/opt/nec/clusterpro/bin/clpmongow.js as follows.

	var DATABASE_NAME       = 'clpmongow';
	var COLLECTION_NAME     = 'monitorCollection';

	print("[I] >>>> START");
	var db = new Mongo().getDB(DATABASE_NAME);
	var col = db.getCollection(COLLECTION_NAME);

	if ( col.find().length() === 1 ) {
			// Normal, do nothing
			print ( "[I] col.find().length() = [" + col.find().length() + "]");
			print ( "[I] GLE = " + db.getLastError() );
	} else if (col.find().length() === 0 ) {
			// Normal, insert document to DB
			print ( "[I] collection [" + COLLECTION_NAME + "] has nothing. Inserting data." );
			var result = col.insert({date: "2015.02.11"});
			print ( "[I] nInserted = " + result.nInserted);
			if (result.nInserted != 1) {
					print ( "[E] GLE = " + db.getLastError() );
			} else {
					print ( "[I] GLE = " + db.getLastError() );
					print ( "[I] a data was inserted to collection [" + COLLECTION_NAME + "]" );
			}
	} else {
			// Abnormal.
			print("[E] <<<< END abnormal : col.find().length() = [" + col.find().length() + "]");
			quit(-1);
	}
	print("[I] <<<< END");
	quit(0);
	
Apply the configuration and start cluster.

2015.02.20 miyamoto KAZuyuki
