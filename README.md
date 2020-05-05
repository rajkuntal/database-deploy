## Current Release Process:
	1. Developer creates database jira ticket & tags it with #Database #pre-deploy/#post-deploy
	2. Ask sysops team to run sql query on QA when it's ready for QA.
	3. Then sysops runs it on stage/prod based on ticket status.
	
## Proposed Release Process:
#### Summary:
	We will have a git repository for database changes. developer can add sql files in there with PR and use jenkins jobs to run sql queries on servers. We can run sql queries through shell scripts(done a POC on that) so It should be doable through jenkins jobs as well.
	   1. Commit new sql files in respective pre-deploy/post-deploy under respective release folder in new branch
	   2. Get PR approved and merge
	   3. Use pre-deploy/post-deploy jenkins jobs to execute sql files on server databases.

## Code hierarchy example: I have created a sample repo in private repository. please take a look : 
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
			post-deploy
				GROWTH-1234.sql
			status.txt


## Required Jenkins Jobs:
### 1. database-release-folder-create-custom:
	This job is to create release folder hierarchy in git if not available. This will avoid merge issues if multiple developers commits same release folders.
### 2. database-deploy: 
	This job is to deploy pre-deploy/post-deploy sql files on a server database. It has two params 1. release-type 2. branch to be deployed.

	A. Git check out given branch in job params
	B. Look for the release version folder as passed in params and run sql files one by one in following steps
		- Get file name(ex: GROWTH-1234)
		- Check if line exists in status.txt for file name based on deploy(pre/post)
			If line exists → check if file is already not executed on current server then run the sql file and append current server name in server list against file name in status.txt(ex: PRE_DEPLOY_GROWTH-1234:QA|STAGE|PROD OR POST_DEPLOY_GROWTH-1234:QA|STAGE|PROD)
			If line not exits → run the sql file and append current server name in server list against file name in status.txt (ex: PRE_DEPLOY_GROWTH-1234:QA|STAGE|PROD OR POST_DEPLOY_GROWTH-1234:QA|STAGE|PROD)

	### These will be 2 types of job for pre-deploy/post-deploy
	### pre-deploy jobs:
		database-pre-deploy-custom → TEAM
			Job params: 
				branch-name : custom
				release-number: custom
		database-pre-deploy-qa → QA
			Job params:
				fixed branch : release
				release-number: custom
		database-pre-deploy-stage → STAGE
			Job params:
				fixed branch : master
				release-number: custom
		database-pre-deploy-prod → PROD
			Job params:
				fixed branch : master
				release-number: custom
				
	### post-deploy jobs:
		database-post-deploy-custom → TEAM
			Job params: 
				branch-name : custom
				release-number: custom
		database-post-deploy-qa → QA
			Job params:
				fixed branch : release
				release-number: custom
		database-post-deploy-stage → STAGE
			Job params:
				fixed branch : master
				release-number: custom
		database-post-deploy-prod → PROD
			Job params:
				fixed branch : master
				release-number: custom
				
## Solution with new git repository and jenkins jobs:
	1. Regular Release: 
	   - Check release folder is available or not if not then create folders using jenkins job 
	   - Create a new branch from develop
	   - Add sql files in respective pre-deploy/post-deploy under respective release folder
	   - Get PR approved and merge into develop branch before release cutman on tuesday to include your changes in release
	   - Use pre-deploy/post-deploy jenkins jobs to execute sql files on server databases.
	2. Rapid Release:
	   - Check rapid release folder is available or not if not then create folders using jenkins job  
	   - create new branch from master
	   - Add sql files in respective pre-deploy/post-deploy under respective release folder
	   - Get PR approved and merge into master
	   - Use pre-deploy/post-deploy jenkins jobs to execute sql files on server databases.
	3. Hot Fix:
	   - Check hotfix folder is available or not if not then create folders using database-release-folder-create-custom  
	   - create new branch from master
	   - Add sql files in respective pre-deploy/post-deploy under respective release folder
	   - Get PR approved and merge into master
	   - Use pre-deploy/post-deploy jenkins jobs to execute sql files on server databases.
