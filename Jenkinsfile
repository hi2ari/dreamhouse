#!groovy

import groovy.json.JsonSlurperClassic

node {

    def SF_CONSUMER_KEY=env.SF_CONSUMER_KEY
    def SF_USERNAME=env.SF_LOGINID
    def SERVER_KEY_CREDENTALS_ID=env.SERVER_KEY_CREDENTALS_ID
    def TEST_LEVEL='RunLocalTests'
    def PACKAGE_NAME='0Ho5w000000KymjCAC'
    def PACKAGE_VERSION='04t5w000003gDRaAAM'  //needs to be removed
    def SF_INSTANCE_URL = env.SF_INSTANCE_URL ?: "https://login.salesforce.com"
    def SFDC_USERNAME = 'test-hhit95g3zydp@example.com'  //needs to be removed
	def RUN_ARTIFACT_DIR="tests\\%BUILD_NUMBER%"
	def SFDC_TESTRUNID

    def toolbelt = tool 'toolbelt'
    
    /***************************************************************************************
    * Variable for capturing the details of the Scratch Org where the package is installed
    ****************************************************************************************/
    def SF_Install_Package_Scratch_Org_Username
    def SF_Install_Package_Scratch_Org_Password
    def SF_Install_Package_Scratch_Org_URL

    // -------------------------------------------------------------------------
    // Check out code from source control.
    // -------------------------------------------------------------------------

    stage('checkout source') {
        checkout scm
    }


    // -------------------------------------------------------------------------
    // Run all the enclosed stages with access to the Salesforce
    // JWT key credentials.
    // -------------------------------------------------------------------------
    
    withEnv(["HOME=${env.WORKSPACE}"]) {
        
        withCredentials([file(credentialsId: SERVER_KEY_CREDENTALS_ID, variable: 'server_key_file')]) {

            // -------------------------------------------------------------------------
            // Authorize the Dev Hub org with JWT key and give it an alias.
            // -------------------------------------------------------------------------

            stage('Authorize DevHub') {
                
            /*    //Logout of any previous account
                rc00 = bat returnStatus: true, script: "\"${toolbelt}\" force:auth:logout --targetusername learn31createorg@gmail.com --noprompt"
                if (rc00 != 0) {
                    error 'logout error.'
                }
                
                //When getting the server.key error for scratch org creation
                //rc0 = bat returnStatus: true, script: "\"${toolbelt}\" force:auth:logout --targetusername ${SF_USERNAME} --noprompt"
                //if (rc0 != 0) {
                //    error 'logout error.'
                //}
             */  
                rc = bat returnStatus: true, script: "\"${toolbelt}\" force:auth:jwt:grant --instanceurl ${SF_INSTANCE_URL} --clientid ${SF_CONSUMER_KEY} --username ${SF_USERNAME} --jwtkeyfile \"${server_key_file}\" --setdefaultdevhubusername --setalias DevHub"//--instanceurl https://login.salesforce.com"
                if (rc != 0) {
                    error 'Salesforce dev hub org authorization failed.'
                }
                //make it the default dev hub
                rc1 = bat returnStatus: true, script: "\"${toolbelt}\" force:config:set defaultusername=${SF_USERNAME} defaultdevhubusername=${SF_USERNAME} --global"
                 if (rc1 != 0) {
                    error 'Salesforce dev hub org is not the default org'
                }
                rc2 = bat returnStatus: true, script: "\"${toolbelt}\" force:org:list"
                if (rc2 != 0) {
                    error 'no list found'
                }
            }
/*

            // -------------------------------------------------------------------------
            // Create new scratch org to test your code.
            // -------------------------------------------------------------------------

            stage('Create Test Scratch Org') {
                rmsg = bat returnStatus: true, script: "\"${toolbelt}\" force:org:create --targetdevhubusername ${SF_USERNAME} --setdefaultusername --definitionfile config/project-scratch-def.json --json  --wait 10 --durationdays 1"
                printf rmsg
				//println('Hello from a Job DSL script1!')
				def beginIndex = rmsg.indexOf('{')
				def endIndex = rmsg.indexOf('}')
				def jsobSubstring = rmsg.substring(beginIndex)
				//println(jsobSubstring)
				def jsonSlurper = new JsonSlurperClassic()
				def robj = jsonSlurper.parseText(jsobSubstring)
				if (robj.status != 0) { error 'org creation failed: ' + robj.message }
				SFDC_USERNAME=robj.result.username
				robj = null
				
            }


            // -------------------------------------------------------------------------
            // Display test scratch org info.
            // -------------------------------------------------------------------------

            stage('Display Test Scratch Org') {
                rc = bat returnStatus: true, script: "\"${toolbelt}\" force:org:display --targetusername ${SFDC_USERNAME}"
                if (rc != 0) {
                    error 'Salesforce test scratch org display failed.'
                }
            }

            // -------------------------------------------------------------------------
            // Push source to the test scratch org.
            // -------------------------------------------------------------------------

            stage('Display Test Scratch Org') {
                rc = bat returnStatus: true, script: "\"${toolbelt}\" force:source:push --targetusername ${SFDC_USERNAME}"
                if (rc != 0) {
                    error 'Salesforce push source to test scratch org failed.'
                }
            }


            // -------------------------------------------------------------------------
            // Assign DreamHouse permission set to scratch org default user.
            // -------------------------------------------------------------------------

            stage('Assign Permset') {
                rc = bat returnStatus: true, script: "\"${toolbelt}\" force:user:permset:assign --targetusername ${SFDC_USERNAME} --permsetname Dreamhouse"
                if (rc != 0) {
                    error 'Salesforce assigning permset Dreamhouse to the defualt user failed.'
                }
            }

            // -------------------------------------------------------------------------
            // Add sample data into app.
            // -------------------------------------------------------------------------

            stage('Import Data') {
                rc = bat returnStatus: true, script: "\"${toolbelt}\" force:data:tree:import --plan data/sample-data-plan.json "
                if (rc != 0) {
                    error 'Salesforce Importing data into the app failed.'
                }
            }


            // -------------------------------------------------------------------------
            // Run unit tests in test scratch org.
            // -------------------------------------------------------------------------

            stage('Run Tests In Test Scratch Org') {
                    bat "mkdir ${RUN_ARTIFACT_DIR}"
                    timeout(time: 120, unit: 'SECONDS') {
                    rmsg = bat returnStdout: true, script: "\"${toolbelt}\" force:apex:test:run --testlevel ${TEST_LEVEL} --outputdir ${RUN_ARTIFACT_DIR} --resultformat tap --json --codecoverage --targetusername ${SFDC_USERNAME}"
                }
                //println('Hello from a Job DSL script4!')
				println(rmsg)
				def beginIndex = rmsg.indexOf('{')
				def endIndex = rmsg.indexOf('}')
				def jsobSubstring = rmsg.substring(beginIndex)
				//println(jsobSubstring)
				def jsonSlurper = new JsonSlurperClassic()
				def robj = jsonSlurper.parseText(jsobSubstring)
				if (robj.status != 0) { error 'Apex test run failed: ' + robj.message }
				SFDC_TESTRUNID = robj.result.summary.testRunId
				println SFDC_TESTRUNID
				robj=null
            }

			// -------------------------------------------------------------------------
            // Generate unit test report.
            // -------------------------------------------------------------------------

            stage('Delete Test Scratch Org') {
                rc = bat returnStatus: true, script: "\"${toolbelt}\" force:apex:test:report -i ${SFDC_TESTRUNID} --resultformat human"
                if (rc != 0) {
                    error 'Salesforce test report generation failed.'
                }
            }

			// -------------------------------------------------------------------------
            // Collect Results.
            // -------------------------------------------------------------------------
			stage('Collect results') {
*/			//	junit keepLongStdio: true, testResults: 'tests/**/*-junit.xml'
/*			}
			
            // -------------------------------------------------------------------------
            // Delete test scratch org.
            // -------------------------------------------------------------------------

            stage('Delete Test Scratch Org') {
				SFDC_USERNAME=null
                rc = bat returnStatus: true, script: "\"${toolbelt}\" force:org:delete --targetusername ${SFDC_USERNAME} --noprompt"
                if (rc != 0) {
                    error 'Salesforce test scratch org deletion failed.'
                }
            }
            */

  /*          // -------------------------------------------------------------------------
            // Create package version.
            // -------------------------------------------------------------------------

            stage('Create Package Version') {
                if (isUnix()) {
                    output = sh returnStdout: true, script: "${toolbelt}/sfdx force:package:version:create --package ${PACKAGE_NAME} --installationkeybypass --wait 10 --json --targetdevhubusername ${SF_USERNAME}"
                } else {
                    output = bat(returnStdout: true, script: "\"${toolbelt}\" force:package:version:create --package ${PACKAGE_NAME} --installationkeybypass --wait 10 --json --targetdevhubusername ${SF_USERNAME}").trim()
                    output = output.readLines().drop(1).join(" ")
                }

                // Wait 5 minutes for package replication.
                sleep 300

                def jsonSlurper = new JsonSlurperClassic()
                def response = jsonSlurper.parseText(output)

                PACKAGE_VERSION = response.result.SubscriberPackageVersionId
                println PACKAGE_VERSION
                response = null
            }


            // -------------------------------------------------------------------------
            // Create new scratch org to install package to.
            // -------------------------------------------------------------------------

            stage('Create Package Install Scratch Org') {
                rmsg = bat returnStdout: true, script: "\"${toolbelt}\" force:org:create --targetdevhubusername ${SF_USERNAME} --setdefaultusername --targetusername ${SFDC_USERNAME} --definitionfile config/project-scratch-def.json --wait 10 --durationdays 1 --json"
                printf rmsg
				//println('Hello from a Job DSL script1!')
				def beginIndex = rmsg.indexOf('{')
				def endIndex = rmsg.indexOf('}')
				def jsobSubstring = rmsg.substring(beginIndex)
				//println(jsobSubstring)
				def jsonSlurper = new JsonSlurperClassic()
				def robj = jsonSlurper.parseText(jsobSubstring)
				if (robj.status != 0) { error 'Scratch org creation for package install failed: ' + robj.message }
				SFDC_USERNAME=robj.result.username
				robj = null
            }


            // -------------------------------------------------------------------------
            // Display install scratch org info.
            // -------------------------------------------------------------------------

            stage('Display Install Scratch Org') {
                rc = bat returnStatus: true, script: "\"${toolbelt}\" force:org:display --targetusername ${SFDC_USERNAME}"
                if (rc != 0) {
                    error 'Salesforce package install scratch org display failed.'
                }
            }

			// -------------------------------------------------------------------------
            // Display package list.
            // -------------------------------------------------------------------------

            stage('Display package list') {
                rc = bat returnStatus: true, script: "\"${toolbelt}\" force:package:list"
                if (rc != 0) {
                    error 'Salesforce package list not displayed.'
                }
            }

*/
            // -------------------------------------------------------------------------
            // Install package in scratch org.
            // -------------------------------------------------------------------------

            stage('Install Package In Scratch Org') {
                rc = bat returnStatus: true, script: "\"${toolbelt}\" force:package:install --package ${PACKAGE_VERSION} --targetusername ${SFDC_USERNAME} --wait 10 --publishwait 10 --noprompt"
                if (rc != 0) {
                    error 'Salesforce package install failed.'
                }
            }

			// -------------------------------------------------------------------------------
            // Assign DreamHouse permission set to package install scratch org default user.
            // -------------------------------------------------------------------------------

            stage('Assign Permset Install Package Scratch Org') {
                //rc = bat returnStatus: true, script: "\"${toolbelt}\" force:user:permset:assign --targetusername ${SFDC_USERNAME} --permsetname Dreamhouse"
                rc = bat returnStatus: true, script: "\"${toolbelt}\" force:user:permset:assign --permsetname Dreamhouse"
                if (rc != 0) {
                    error 'Salesforce assigning permset Dreamhouse to the package install scratch org defualt user failed.'
                }
            }

            // -------------------------------------------------------------------------
            // Add sample data into package install scratch org app.
            // -------------------------------------------------------------------------

            stage('Import Data') {
                rc = bat returnStatus: true, script: "\"${toolbelt}\" force:data:tree:import --plan data/sample-data-plan.json "
                if (rc != 0) {
                    error 'Salesforce Importing data into the package install scratch org app failed.'
                }
            }

            // -------------------------------------------------------------------------
            // Run unit tests in package install scratch org.
            // -------------------------------------------------------------------------

            stage('Run Tests In Package Install Scratch Org') {
                rc = bat returnStatus: true, script: "\"${toolbelt}\" force:apex:test:run --targetusername ${SFDC_USERNAME} --resultformat human --codecoverage --testlevel ${TEST_LEVEL} --wait 10"
                if (rc != 0) {
                    error 'Salesforce unit test run in pacakge install scratch org failed.'
                }
            }


            // -------------------------------------------------------------------------
            // Generate password for the user of package install scratch org user.
            // -------------------------------------------------------------------------

            stage('Generate Password for Package Install Scratch Org User') {
                rmsg = bat returnStdout: true, script: "\"${toolbelt}\" force:user:password:generate --targetusername ${SFDC_USERNAME} --json"
                //if (rc != 0) {
                //   error 'Salesforce password generatation for package install scratch org user failed.'
                //}
				println(rmsg)
				//println('Hello from a Job DSL script1!')
				def beginIndex = rmsg.indexOf('{')
				def endIndex = rmsg.indexOf('}')
				def jsobSubstring = rmsg.substring(beginIndex)
				println(jsobSubstring)
				def jsonSlurper = new JsonSlurperClassic()
				def robj = jsonSlurper.parseText(jsobSubstring)
				if (robj.status != 0) { error 'Salesforce password generatation for package install scratch org user failed: ' + robj.message }
				
            }
			
			// -------------------------------------------------------------------------
            // Display details for the user of package install scratch org user.
            // -------------------------------------------------------------------------
			
			stage('Display Details for Package Install Scratch Org User') {
				rmsg = bat returnStdout: true, script: "\"${toolbelt}\" force:user:display --targetusername ${SFDC_USERNAME} --json"
				println(rmsg)
				//println('Hello from a Job DSL script1!')
				def beginIndex = rmsg.indexOf('{')
				def endIndex = rmsg.indexOf('}')
				def jsobSubstring = rmsg.substring(beginIndex)
				println(jsobSubstring)
				def jsonSlurper = new JsonSlurperClassic()
				def robj = jsonSlurper.parseText(jsobSubstring)
				if (robj.status != 0) { error 'Display details for package install scratch org user failed: ' + robj.message }
				//SFDC_USERNAME=robj.result.username
                SF_Install_Package_Scratch_Org_Username = robj.result.username
                SF_Install_Package_Scratch_Org_Password = robj.result.password
                SF_Install_Package_Scratch_Org_URL = robj.result.instanceUrl
				robj = null
			}
        }
    }
}


//Setting the value needs to be deleted after checking this build
env.SF_Install_Package_Scratch_Org_Username = SF_Install_Package_Scratch_Org_Username
env.SF_Install_Package_Scratch_Org_Password = SF_Install_Package_Scratch_Org_Password
env.SF_Install_Package_Scratch_Org_URL = SF_Install_Package_Scratch_Org_URL

def command(script) {
    if (isUnix()) {
        return sh(returnStatus: true, script: script);
    } else {
        return bat(returnStatus: true, script: script);
    }
}
