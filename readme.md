# SDP - error handing of the communication between Cloudant&dashDB

This document introduces how to amend the communicate error between Cloudant&dashDB when they are both on dedicated environment which can be a playbook for Cloudant service team.

The listed error codes and messages can be found in either of

* the _warehouser document
* the _overflow table
* the log files
* the dashboard

Below are the runtime problems that should be fixed by support/operations.

## Symptom-from dashboard

### 1.Error Code=40004 when type the 'create warehouse' icon on the bottom of 'Warehouses/Create a Warehouse' page

**A typical error message**: Error:Cannot connect to database.[[statusCode=40000,errorCode=40004,errorMsg=Cannot connect to database.
[com.ibm.db2/cc.am.DisconnectNonTransientConnectionException:[jcc][t4][2043][11550][4.18.60] Error opening socket to server dashdb-enterprise*.bluemix.net on port * with message: Connection timed out.ERRORCODE=-4499, SQLSTATE=08001

**Cause**: The dashDB console is down or having trouble; in certain cases such as DB2 down but dashDB console up you can get this even if the dashDB endpoint returns OK

**Solution**: Test if you can connect to the dashDB URL.

`acrul https://<cloudant-user>.cloudant.com/_warehouser/<_id>` to get the URL in 
`"dashboard_url"` with user id in `"dynamite_user"` and password in `"dynamite_token"`

*Note*: You may have to truncate the URL to reduce to https://<server>:<port>/console/ibmblu only
 
If the dashDB console does not respond or the tables don't load under 'Manage / Work with Tables' contact the dashDB support to resolve this issue. This is not an SDP problem. 
