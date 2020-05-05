## Code hierarchy 
	database-deploy
		262
		262.4
		262-week
		263
		263.2
		263-week
		264
			pre-deploy
				GROWTH-1234.sql
				TRADE-3798.sql
				status.properties
			post-deploy
				SELLERTOOLS-6590.sql
				status.properties
		version.txt
		rapid-version.txt
		hotfix-version.txt


## Required Jenkins Jobs:
### 1. database-release-folder-create-custom: 
	This job will be used by developers to create folder hierarchy with status.properties file for any release in master branch(always), it takes the release number as a param. example: developer has a ticket of 112 release with database changes and if he/she wanted to commit sql files then he/she should check if folder hierarchy is available or not for 112 release
	
	if yes → then simply he/she can create new sql file under respective folder(pre-deploy/post-deploy)
	
	If no → then he/she should run this job by entering release number(112)
	
	Same process applies for rapid release and hot-fix.

### 2. database-deploy: 
	This job is to deploy pre-deploy/post-deploy sql files on a server database. It has two params 1. release-type 2. branch to be deployed.

	A. Git check out given branch in job params
	B. Pick the release version from *.properties file based on release-type. 
	C. Look for the release version folder then check the status in status.properties inside the pre-deploy/post-deploy folder. Since changes can be deployed multiple times on team/qa/stage servers so status check will happen only in production release job.
	D. If prod job & status is not completed OR non-prod job then run sql files one by one in following steps
		I. Get file name(ex: GROWTH-1234)
		II. Check if line exists in status.properties for file name
			If yes → get the server list where file is already executed and check if current server is already present in list if yes then don’t run the file else run the file and append current server name in server list against file name in status.properties (ex: GROWTH-1234:QA|STAGE)
			If No → run the file and append current server name in server list against file name in status.properties
	E. If prod release then add a new line in status.properties with release completed status. (ex: RELEASE:COMPLETED)

	### These will be 2 types of job for pre-deploy/post-deploy
	### pre-deploy jobs:
		database-pre-deploy-custom → TEAM
			Job params: 
				release-version : 262
				branch-name : develop
		database-pre-deploy-qa → QA
			Job params:
				fixed branch : release
				release-type: regular/rapid/hotfix
		database-pre-deploy-stage → STAGE
			Job params:
				fixed branch : master
				release-type: regular/rapid/hotfix
		database-pre-deploy-prod → PROD
			Job params:
				fixed branch : master
				release-type: regular/rapid/hotfix
				
	### post-deploy jobs:
		database-post-deploy-custom → TEAM
			Job params: 
				release-version : 262
				branch-name : develop
		database-post-deploy-qa → QA
			Job params:
				fixed branch : release
				release-type: regular/rapid/hotfix
		database-post-deploy-stage → STAGE
			Job params:
				fixed branch : master
				release-type: regular/rapid/hotfix
		database-post-deploy-prod → PROD
			Job params:
				fixed branch : master
				release-type: regular/rapid/hotfix
## Current Release Process:
	1. Developer creates database jira ticket & tags it with #Database #pre-deploy/#post-deploy
	2. Ask sysops team to run sql query on QA when it's ready for QA.
	3. Then sysops runs it on stage/prod based on ticket status.
	
## Proposed Release Process:
	1. Regular Release: 
	   - Check release folder is available or not if not then create folders using database-release-folder-create-custom 
	   - Create a new branch from develop
	   - Add sql files in respective pre-deploy/post-deploy under respective release folder
	   - Get PR approved and merged into develop branch before release cutman on tuesday to include your changes in release
	2. Rapid Release:
	   - Check rapid release folder is available or not if not then create folders using database-release-folder-create-custom  
	   - create new branch from master
	   - update rapid-version.properties with your rapid release version
	   - Add sql files in respective pre-deploy/post-deploy under respective release folder
	   - Get PR approved and merged in master
	3. HotFix:
	   - Check hotfix folder is available or not if not then create folders using database-release-folder-create-custom  
	   - create new branch from master
	   - update hotfix-version.properties with your hotfix version
	   - Add sql files in respective pre-deploy/post-deploy under respective release folder
	   - Get PR approved and merged in master
