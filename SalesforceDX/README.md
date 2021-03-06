<div style="text-align:right;top: 10px;position: absolute;right: 10px;" markdown="1">
	<img align="right" src="http://www.smsmt.com/hs-fs/hubfs/SMS_Logo-1.png?t=1490163156935&amp;width=300&amp;name=SMS_Logo-1.png"/>
</div>

# SalesforceDX #
## Manage Scratch Orgs ##
--------------------------
Create a scratch org for development using a scratch org definition file<br/>
	```sfdx force:org:create -f config/project-scratch-def.json```<br/>

Specify scratch org definition values on the command line using key=value pairs<br/>
	```sfdx force:org:create adminEmail=david.browaeys@smsmt.com edition=Developer```<br/>

Create a scratch org with an alias (this will provide a autogenerated username)<br/>
	```sfdx force:org:create -f project-scratch-def.json -a MyScratchOrg```<br/>

Indicate that this scratch org is the default<br/>
	```sfdx force:org:create -f project-scratch-def.json --setdefaultusername```<br/>

Open the org. in default Browser:<br/>
	```sfdx force:org:open -u <username/alias>```<br/>

Open the org in Lightning Experience or open a Visualforce page, use the --path parameter.<br/>
	```sfdx force:org:open --path one/one.app```<br/>
	```sfdx force:org:open --path apex/MyVisualforcePage```<br/>

Important note: a scratch org will be deleted after 7 days. <br/>
## Initialise scratch org ##
--------------------------
Retrieve metadata from an org(use mainly when retrieving metadata from different org):<br/>
	```sfdx force:mdapi:retrieve -s -r ./mdapipkg -u <username> -p <package name>```<br/>

Unzip file:<br/>
	```unzip mdapipkg/unpackaged.zip```

Convert metadata API source to SFDX project<br/>
	```sfdx force:mdapi:convert --rootdir unpackaged/```

Push change: push local changes to scratch org<br/>
		```sfdx force:source:push -u <username/alias>```
		
## Manage changes between scratch orgs ##
--------------------------
Local files will be sync with org changes. <br/>
	Check change status<br/>
		```sfdx force:source:status```

	Pull change: to get changes from org<br/>
		```sfdx force:source:pull -u <username/alias>```

	Push change: push local changes to org<br/>
		```sfdx force:source:push -u <username/alias>```

## Deploy change to sandbox (non scratch org) ##
--------------------------
This represents a full deployment<br/>
Pull change: make sure you have all changes from scratch org<br/>
	```sfdx force:source:pull -u <username/alias>```

Create output folder:<br/>
	```mkdir mdapi_output_dir```

Convert  SFDX Project source to metadata<br/>
	```sfdx force:source:convert -d mdapi_output_dir/ --packagename package_name```

Deploy metadata to target org<br/>
	```sfdx force:mdapi:deploy -d mdapi_output_dir/ -u "sandbox_username" -l RunSpecifiedTests -r test1,test2,test3,test4```

If the deployment job wait time is 1 minute or more, the status messages update every 30 seconds.
If too long, you can check status and ask to wait x minute by using "-w number", i.e.:<br/>
	```sfdx force:mdapi:deploy -u "sandbox_username" -i 0AfB0000009SvyoKAC -w 5```<br/>
where 0AfB0000009SvyoKAC is the job id return with mdapi:deploy command. <br/>

## Manage test coverage ##
--------------------------
Run Test ASYNC:sfdx will return job id. <br/>
	```sfdx force:apex:test:run -u <username/alias>```<br/>

See Test Result : <br/>
	```sfdx force:apex:test:report -i 7072800005SYWIq -u BRO1```<br/>

Run Test SYNC:<br/>
	```sfdx force:apex:test:run --resultformat human```<br/>

## Manage Data ##
-------------------------
You can transfer data from one org to another org. This would be really helpful to setup master data from production to dev sandboxes.<br/>

Make a simple query :<br/>
	```sfdx force:data:soql:query -q "SELECT ID, Name, Status FROM ScratchOrgInfo WHERE CreatedBy.Name = '<your name>' AND CreatedDate = TODAY" -u <org alias>```

Export accounts with contacts : <br/>
	```sfdx force:data:tree:export --query \
      "SELECT Id, Name
       (SELECT Id, FirstName, Lastname\
        FROM Contacts) \
       FROM Account" \
     --prefix export-demo --outputdir sfdx-out --plan -u sms-partner```

SFDX will generate a reference for all recorcid, i.e.: AccountRef1, ContactRef1, ... So when exporting account and contact in same time, SFDX can maintain the relationship between both objects, example : 
```json
	{
            "attributes": {
                "type": "Contact",
                "referenceId": "ContactRef19"
            },
            "FirstName": "Jake",
            "LastName": "Llorrac",
            "AccountId": "@AccountRef12"
        }
```

Import account with contacts<br/>
 ```sfdx force:data:tree:import --targetusername sms-partner2 \
    --plan sfdx-out/export-demo-Account-Contact-plan.json ```
