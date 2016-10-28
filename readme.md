# SDP - error handing of the communication between dedicated Cloudant&dashDB

This document introduces how to amend the communicate error between Cloudant&dashDB when they are both on dedicated environment.

## Symptoms

### 1. Error message come out when click ‘Analytics’in the panel on the left side of Cloudant dashboard.

    If you don't already have a dashDB instance, you can obtain one on IBM Bluemix.
    If you already have a dashDB instance, contact us at support@cloudant.com and we will enable the warehouse feature for your account.

### 2. Error message come out shortly after that.
 
    dashDB Error
    Sorry, dashDB is down. Click the button below to try to reconnect.
       
### 3. Try this command, the output says:"DASHDB":"down".
 
`acurl https://<account_name>.cloudant.com/_api/v2/partners/dashdb/warehouse/service_status`

An example of the response for the above command:
```json
{
  "SDP_HOST": "sdp-wdc01-1.cloudant.com:443", 
  "SDP": "ok", 
  "REGION": "us-south", 
  "DASHDB": "down", 
  "DASHDB_HOST": "dashdb-enterprise4-iadb-wdc04-01.services.dal.bluemix.net"
}
```

## Solutions

### 1. Change the dashDB region if the region mapping is invalid.

Query the region mapping for a specific cluster:

`acurl https://dashdb-cloudant-regions.cloudant.com/region_definitions/<cluster_name>`

An example of the response for the above command:
```json
{
  "_id": "...",
  "region": "us-south",
  "cloudant_clusters": [
    "..."
  ],
  "dashdb_invalid": "https://dashdb-enterprise-yp-lon02-03.services.eu-gb.bluemix.net",
  "sdp": "https://sdp-wdc01-1.cloudant.com:443",
...
}
```

Each entry in a region config doc will have a field for the 'dashdb url', keyed on 'dashdb'. This URL should be pointing to the dashDB service API. Currently there are only four valid entries:
  - US-S: https://dashdbrm.ng.bluemix.net
  - EU: https://dashdbrm.eu-gb.bluemix.net
  - AU-Syd: https://dashdbrm.au-syd.bluemix.net
  - test010 : https://dashdbrm-testy.mybluemix.net

There have been a few instances of the incorrect URL being used, likely the user passing it on (from the dashDB dashboard) through sales/support. These URLs look like so:
  - https://dashdb-enterprise-yp-lon02-03.services.eu-gb.bluemix.net

This type of URL is used to access the dashDB console, DB2 instances, etc. It is not the correct URL to use for the integration configuration. If meet this, fix it by changing `"dashdb_invalid":"https://dashdb-enterprise-yp-lon02-03.services.eu-gb.bluemix.net"` to `"dashdb": "https://dashdbrm.ng.bluemix.net"` for instance.

### 2. Activate this enterprise user by this command.

`clou user dashdb --set_enterprise <account_name>`

An example of the response for the above command:
```json
Current dashdb feature data:
{
    "dashdb": null
}

Add dashdb feature response:
{
    "ok": true, 
    "feature": "dashdb"
}
```

*Note*: After this, customer should see 'create warehouse' icon from dashboard instead of 'dashDB error' again.

## Symptoms

### Error message come out when type the 'create warehouse' icon on the bottom of 'Warehouses/Create a Warehouse' page

**A typical error message**: Error:Cannot connect to database.[[statusCode=40000,errorCode=40004,errorMsg=Cannot connect to database.
[com.ibm.db2/cc.am.DisconnectNonTransientConnectionException:[jcc][t4][2043][11550][4.18.60] Error opening socket to server dashdb-enterprise*.bluemix.net on port * with message: Connection timed out.ERRORCODE=-4499, SQLSTATE=08001

## Solutions

### 1. Tell the customer that they need to create an official request to dashDB team with their own approval of opening the relevant firewalls.

### 2. Find the IP addresses assosiated with the customer from SDP clusters, then work with someone in #cdsni to open the firewall rules.

* a. Find which sdp cluster is being used for a speific user by searching `index="*sdp*" "<account_name>"` in splunk, or just from here:https://dashdb-cloudant-regions.cloudant.com/region_definitions/_design/regions/_view/by_sdp.
* b. Find all nodes in this cluster by `knife node list | grep sdp`.
* c. Log on to these nodes one by one and get the IP addresses from /etc/network/interfaces.(The load balancer will decide every time which node to route the request to in the given cluster. So we have to open every node.)

*Note*: This issue only affects customers that have both a dedicated Cloudant and a dedicated dashDB environment. For everyone else the integration works without firewall changes.
