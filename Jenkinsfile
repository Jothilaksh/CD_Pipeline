import java.text.SimpleDateFormat
def dateFormat
def date
def formattedDate
def public approverusers
def public emailusers
int count;
def rel_team_user="gopinath_t@solartis.com,jothilakshmi_m@solartis.com,karthickraja_m@solartis.com,pavithradevi_p@solartis.com,rajarathinasabapathi_s@solartis.com,ranjithkumar_s@solartis.com,shyamsundar_r@solartis.com"

def current_time()
{
	dateFormat = new SimpleDateFormat("HH:mm:ss")
	date = new Date()
	sh "date +'%T' > result";
    formattedDate=readFile('result').trim()

}

def package_initial_process()
{
echo "Packaging Phase: "
//sh '/usr/bin/solartis/CD_Automation/SDPFileCreationV2/packagingV4.sh ${Customer} ${LOB} ${Support_Ticket} ${Release_Number}' 
//sh '/usr/bin/solartis/CD_Automation/SDPFileCreationV2/jsoninfoV2.sh ${Customer} ${LOB} ${Support_Ticket} ${Release_Number}'
}

def package_upload()
{
echo "PackageUpload"
//sh '/usr/bin/solartis/CD_Automation/SDPFileCreationV2/PackageUpload.sh ${Customer} ${LOB} ${Support_Ticket} ${Release_Number}'
}

def release_info()
{
echo "RELEASE INFO \n ================== \n Release Customer: ${Customer}\n Customer LOB:  ${LOB} \n Release_Support Ticket: ${Support_Ticket} \n RELEASE NUMBER: ${Release_Number}"
}

def stages_initial_Process()
{
//sh '/usr/bin/solartis/CD_Automation/PackDownload/InitialFolderChk.sh ${Support_Ticket}-${Release_Number}'
//sh 'cd /opt/CD/ && curl -X GET -H "Authorization: Basic YWRtaW46YWRtaW4xMjM=" "http://192.168.85.154:8081/nexus/service/local/repositories/SolartisDeploymentPackage/content/${Customer}/${LOB}/${Support_Ticket}/${Release_Number}/${Support_Ticket}-${Release_Number}.sdp" -O --output /opt/CD/'
//sh 'unzip -o /opt/CD/${Support_Ticket}-${Release_Number}.sdp -d /opt/CD/${Support_Ticket}-${Release_Number}'
//sh '/usr/bin/solartis/CD_Automation/PackDownload/InitialStatusEntryV4.sh ${Support_Ticket}-${Release_Number}'
}

def sql_backup_execution()
{
//sh '/usr/bin/solartis/CD_Automation/SQL/sqlbackup_V2.sh ${Support_Ticket}-${Release_Number} '
//sh '/usr/bin/solartis/CD_Automation/SQL/sqlexecution_V2.sh ${Support_Ticket}-${Release_Number} '
}

def kb_deploy()
{
//sh '/usr/bin/solartis/CD_Automation/KnowledgeBase/kbdeployV5.sh ${Support_Ticket}-${Release_Number}'				
}
def form_deploy()
{
//sh '/usr/bin/solartis/CD_Automation/Forms/formsdeployV4.sh ${Support_Ticket}-${Release_Number}'
}
def noderestart()
{
//sh '/usr/bin/solartis/CD_Automation/KnowledgeBase/NodeRestartCondition.sh ${Support_Ticket}-${Release_Number}'
}

def approveuser_list(String approve_user_name)
{
				sh "/usr/bin/solartis/CD_Automation/Users/approveuser.sh ${approve_user_name} > approverslist"
				approverusers = readFile('approverslist').trim()
				echo approverusers
				return approverusers
}

def userEmailList()
{
sh "/usr/bin/solartis/CD_Automation/Users/emailuser.sh ${LOB} > emailuserslist"
emailusers = readFile('emailuserslist').trim()
return emailusers
}

pipeline {
     agent none
     parameters{
choice(name: 'Customer',choices:'Starr',description: 'Select Customer')
choice(name: 'LOB', choices:'LGL', description: 'select project')
string(name: 'Support_Ticket', defaultValue: 'LGL_DocumentGeneration', description: 'Enter Release Version without space')
string(name: 'Release_Number', defaultValue: 'TEST1234', description: 'Enter Ticket Number')
}  
    stages {
            stage('Packaging') {
            agent { node {  label 'ConfigTier'  } }
            steps {
               script {
			   try{
                   count =0;
                   wrap([$class: 'BuildUser']) {
                   currentBuild.description = "${Release_Number}-${Support_Ticket} triggered by ${BUILD_USER}"
                   }
				release_info()
				package_initial_process()
				echo "${LOB}_DEV"
				sh "/opt/temp.sh ${BUILD_NUMBER}"
				env.RELEASE_PKGAPPROVAL = input(message: 'Should We Proceed GOT', ok: 'Yes', 
                        parameters: [booleanParam(defaultValue: true, 
                        description: 'Group Of Ticktes Release',name: 'Yes?')])
                approverusers=approveuser_list("${LOB}_DEV")				
				env.RELEASE_PKGAPPROVAL = input id: 'PackageVerification', message: 'Package verification', ok: 'Release!', submitter: approverusers,
                parameters: [choice(name: 'RELEASE_SCOPE', choices: 'NO\nYES', description: 'Promote Package to proceed Alpha Release')]
				if (env.RELEASE_PKGAPPROVAL == "YES") {
				package_upload()
				}

			}
			catch(Exception e){
                echo "${e}"
				retry(10){
				echo "Skipped mail"
				//emailusers=userEmailList("${LOB}")
				if(count == 0){
                //mail(from: "ReleaseUpdates@solartis.com", 
                            // to: emailusers, 
							// cc: rel_team_user,	
                            // subject: "Release-${customer}-${Release_Number}",
                            // body: "Packaging Failed  for ticket  ${Release_Number} \n Fix the issue and click Rerun \n Kindly check log in below link:\nFor blue ocean view: ${RUN_DISPLAY_URL}\nFor classic view: ${BUILD_URL}" )                     			   
			} 
			else{
			echo "Skipped mail"
			//mail(from: "ReleaseUpdates@solartis.com", 
             //                to: emailusers, 
			//	/			 cc:rel_team_user,	
                 //            subject: "Release-${customer}-${Release_Number}",
                   //          body: "Packaging Again Failed in  for ticket  ${Release_Number} \n Fix the issue and click Rerun \nRetry count:${count}\n Kindly check log in below link:\nFor blue ocean view: ${RUN_DISPLAY_URL}\nFor classic view: ${BUILD_URL}" )
			}
			approverusers = approveuser_list("${LOB}_DEV")
			env.RELEASE_SCOPE = input id: 'ReRunPackaging', message: 'Task execution  Failed in Packaging  Once issue Fixed Leave Comment and rerun', ok: 'RERUN!', submitter: approverusers,
                            parameters: [string(name: 'Approval_string', defaultvalue: '', description: 'Approval comment:')]  
			count =count+1;
			echo "RePackaging for ${count} time:"
               package_initial_process()
			   approverusers=approveuser_list("${LOB}_DEV")
				env.RELEASE_PKGAPPROVAL = input id: 'PackageVerificationReRun', message: 'Package verification', ok: 'Release!', submitter: approverusers,
                            parameters: [choice(name: 'RELEASE_SCOPE', choices: 'NO\nYES', description: 'Promote Package to proceed Alpha Release')]
                if (env.RELEASE_PKGAPPROVAL == "YES") { 			
					package_upload()
				}
			}
			}
            }
        }
			post
              {
                 always{
                     script
                     {
                             echo "Skipped mail"
					         //emailusers=userEmailList()							
                             //mail(from: "ReleaseUpdates@solartis.com", 
                            // to: emailusers, 
							// cc:rel_team_user,	
                            // subject: "Release-${customer}-${Release_Number}",
                            // body: "Pacakging Activity Completed for ticket  ${Release_Number}\n Entering into alpha\nKindly check log in below link:\nFor blue ocean view: ${RUN_DISPLAY_URL}\nFor classic view: ${BUILD_URL} " )
							 
                 }
             }
	     }
	}        
		
        stage('Alpha') {
            agent { node {  label 'ConfigTier'  } }
            steps {
                script {
                    try{
                    count =0;
                     wrap([$class: 'BuildUser']) {
                    currentBuild.description = "${Release_Number}-${Support_Ticket} triggered by ${BUILD_USER}"
                            echo "Skipped mail"
                              //emailusers = userEmailList()
				   			 //mail(from: "ReleaseUpdates@solartis.com", 
                             //to: emailusers, 
							 //cc:rel_team_user,	
                             //subject: "Release-${customer}-${Release_Number}",
                             //body: "Deployment Started for ${Release_Number} in Alpha \n Release Started by: ${BUILD_USER} \n click below link to check log:\n For blue ocean view: ${RUN_DISPLAY_URL}\nFor classic view: ${BUILD_URL} "  )
                   }
				stages_initial_Process()   
				echo "SQL Execution and Backup..."
				sql_backup_execution()
				echo "Forms Backup and Deployment"
		     	form_deploy() 
				echo "KB Backup and Deployment"
				kb_deploy()
				noderestart()
				approverusers = approveuser_list("ReleaseEngineering")
				env.RELEASE_TESTAPPROVAL = input id: 'RelAlpha', message: 'Release Team Approval Required', ok: 'Done!', submitter: approverusers,
                parameters: [choice(name: 'TestVerificationApproval', choices: 'NO\nYES', description: 'Release Verification Test Completed')]

                if (env.RELEASE_TESTAPPROVAL == "NO") {
					env.RELEASE_TESTAPPROVAL = input id: 'RollBackAlpha', message: 'Rollback Approval Required', ok: 'Done!', submitter: approverusers,
                parameters: [choice(name: 'TestVerificationApproval', choices: 'NO\nYES', description: 'Approve to Rollback')]
				
				if (env.RELEASE_TESTAPPROVAL == "YES") {
					echo "Rollback"
					//sh '/usr/bin/Solartis/CD_Automation/Rollback/rollbackv2.sh ${Support_Ticket}-${Release_Number}'
				}
                }
                }
			
                catch(Exception e){
                    		echo "${e}"
				retry(10){
				
				if(count == 0){
				    echo "Skipped mail"
                //mail(from: "ReleaseUpdates@solartis.com", 
                     //        to: emailusers, 
					//		 cc:rel_team_user,	
                         //    subject: "Release-${customer}-${Release_Number}",
                         //    body: "Deployment Failed in Alpha for ticket  ${Release_Number} \n Fix the issue and click Rerun \n Kindly check log in below link:\nFor blue ocean view: ${RUN_DISPLAY_URL}\nFor classic view: ${BUILD_URL}" )                     			   
			} 
			else{
			    echo "Skipped mail"
			//mail(from: "ReleaseUpdates@solartis.com", 
                        //     to: emailusers, 
					//		 cc:rel_team_user,	
                          //   subject: "Release-${customer}-${Release_Number}",
                           //  body: "Deployment Again Failed in ALPHA for ticket  ${Release_Number} \n Fix the issue and click Rerun \nRetry count:${count}\n Kindly check log in below link:\nFor blue ocean view: ${RUN_DISPLAY_URL}\nFor classic view: ${BUILD_URL}" )
			}
				sh "/usr/bin/solartis/CD_Automation/Users/approveuser.sh ${LOB}_DEV > approverslist"
				approverusers = readFile('approverslist').trim()
				echo approverusers
			env.RELEASE_SCOPE = input id: 'ReRunAlpha', message: 'Task execution  failed in ALPHA  once issue fixed Leave Comment and rerun', ok: 'RERUN!', submitter: approverusers,
                            parameters: [string(name: 'Aprroval_string', defaultvalue: '', description: 'Approval comment:')]  
			count =count+1;
	            stages_initial_Process()
				echo "SQL Execution and Backup..."
				sql_backup_execution()
				echo "Forms Backup and Deployment"
		     	form_deploy()
				echo "KB Backup and Deployment"
                kb_deploy()
				noderestart()
				approverusers = approveuser_list("ReleaseEngineering")
				env.RELEASE_TESTAPPROVAL = input id: 'RelAlphaReRun', message: 'Release Team Approval Required', ok: 'Done!', submitter: approverusers,
                parameters: [choice(name: 'TestVerificationApproval', choices: 'NO\nYES', description: 'Release Verification Test Completed')]

                if (env.RELEASE_TESTAPPROVAL == "NO") {
					env.RELEASE_TESTAPPROVAL = input id: 'RollBackAlphaReRun', message: 'Rollback Approval Required', ok: 'Done!', submitter: approverusers,
                parameters: [choice(name: 'TestVerificationApproval', choices: 'NO\nYES', description: 'Approve to Rollback')]
				
				if (env.RELEASE_TESTAPPROVAL == "YES") {
					echo "Rollback"
					//sh '/usr/bin/Solartis/CD_Automation/Rollback/rollbackv2.sh ${Support_Ticket}-${Release_Number}'
				}
				
                }
                
                }       
				}
				
		}
            }
			post
              {
                 always{
                     script
                     {
                         echo "Skipped mail"
						    // emailusers=userEmailList()
                            // mail(from: "ReleaseUpdates@solartis.com", 
                             //to: emailusers, 
							 //cc:rel_team_user,	
                            // subject: "Release-${customer}-${Release_Number}",
                             //body: "Release Success  in alpha for ticket  ${Release_Number} \n To enter Beta Environment Waiting for approvals.Following assignees involved to approve:\nadmin\nbauser\nPM\nKindly check log in below link:\nFor blue ocean view: ${RUN_DISPLAY_URL}\nFor classic view: ${BUILD_URL} " )
							 
					    approverusers = approveuser_list("${LOB}_BA")		 
							env.RELEASE_SCOPE2 = input id: 'AlphaApprovalBA', message: 'BA Approval Required', ok: 'Release!', submitter: approverusers,
                            parameters: [choice(name: 'RELEASE_SCOPE', choices: 'NO\nYES', description: 'Promote BETA Release')]
							if (env.RELEASE_SCOPE2 == "NO") {
							approverusers = approveuser_list("ReleaseEngineering")
								env.RELEASE_TESTAPPROVAL = input id: 'RollbackBA', message: 'Rollback Approval Required', ok: 'Done!', submitter: approverusers,
								parameters: [choice(name: 'TestVerificationApproval', choices: 'NO\nYES', description: 'Need to Rollback')]
				
								if (env.RELEASE_TESTAPPROVAL == "YES") {
									echo "Rollback"
									//sh '/usr/bin/Solartis/CD_Automation/Rollback/rollbackv2.sh ${Support_Ticket}-${Release_Number}'
								}
				
							}
                 }
             }
	     }

        }
        
        stage('Beta') {
		agent { node {  label 'ConfigTier'  } }
            steps {
                script {
				   try{
				   count =0;
                   wrap([$class: 'BuildUser']) {
                   currentBuild.description = "${Release_Number}-${Support_Ticket} triggered by ${BUILD_USER}"
                   }
                stages_initial_Process()
				echo "SQL Execution and Backup..."
                sql_backup_execution()
				echo "Forms Backup and Deployment"
		     	form_deploy()
				echo "KB Backup and Deployment"
				kb_deploy()
				noderestart()
				approverusers=approveuser_list("ReleaseEngineering")
				env.RELEASE_TESTAPPROVAL = input id: 'RelBeta', message: 'Release Team Approval Required', ok: 'Done!', submitter: approverusers,
                parameters: [choice(name: 'TestVerificationApproval', choices: 'NO\nYES', description: 'Release Verification Test Completed')]

                if (env.RELEASE_TESTAPPROVAL == "NO") {
					env.RELEASE_TESTAPPROVAL = input id: 'RollbackRelBeta', message: 'Rollback Approval Required', ok: 'Done!', submitter: approverusers,
                parameters: [choice(name: 'TestVerificationApproval', choices: 'NO\nYES', description: 'Approve to Rollback')]
				
				if (env.RELEASE_TESTAPPROVAL == "YES") {
					echo "Rollback"
					//sh '/usr/bin/Solartis/CD_Automation/Rollback/rollbackv2.sh ${Support_Ticket}-${Release_Number}'
				}
                }
                }
				catch(Exception e)
				{
 				echo "${e}"
				retry(10){
				//emailusers=userEmailList()
				if(count == 0){
				echo "Skipped mail"
                //mail(from: "ReleaseUpdates@solartis.com", 
                 //            to: emailusers, 
				//			 cc: rel_team_user,	
                    //         subject: "Release-${customer}-${Release_Number}",
                     //        body: "Deployment Failed in BETA for ticket  ${Release_Number} \n Fix the issue and click Rerun. Kindly check log in below link:\nFor blue ocean view: ${RUN_DISPLAY_URL}\nFor classic view: ${BUILD_URL}" )
                     			   
			} 
			else{
			    echo "Skipped mail"
			//mail(from: "ReleaseUpdates@solartis.com", 
                 //            to: emailusers, 
					//		 cc: rel_team_user,
                       //      subject: "Release-${customer}-${Release_Number}",
                         //    body: "Deployment Again Failed in BETA for ticket  ${Release_Number} \n Fix the issue and click Rerun Kindly check log in below link:\nFor blue ocean view: ${RUN_DISPLAY_URL}\nFor classic view: ${BUILD_URL}" )
			}
				approverusers=approveuser_list("${LOB}_DEV")
			env.RELEASE_SCOPE = input id: 'ReRunBeta', message: 'Task execution  Failed in BETA  Once issue Fixed Leave Comment and rerun', ok: 'RERUN!', submitter: approverusers,
                            parameters: [string(name: 'Aprroval_string', defaultvalue: '', description: 'Approval comment:')]  
			count =count+1;
				stages_initial_Process()
				echo "SQL Execution and Backup..."
				sql_backup_execution()
				echo "Forms Backup and Deployment"
		     	form_deploy() 
				echo "KB Backup and Deployment"
				kb_deploy()
				noderestart()
				approverusers = approveuser_list("ReleaseEngineering")
				
				env.RELEASE_TESTAPPROVAL = input id: 'RelBetaReRun', message: 'Release Team Approval Required', ok: 'Done!', submitter: approverusers,
                parameters: [choice(name: 'TestVerificationApproval', choices: 'NO\nYES', description: 'Release Verification Test Completed')]

                if (env.RELEASE_TESTAPPROVAL == "NO") {
					env.RELEASE_TESTAPPROVAL = input id: 'RollBackBetaReRun', message: 'Rollback Approval Required', ok: 'Done!', submitter: approverusers,
                parameters: [choice(name: 'TestVerificationApproval', choices: 'NO\nYES', description: 'Approve to Rollback')]
				
				if (env.RELEASE_TESTAPPROVAL == "YES") {
					echo "Rollback"
					roll_back()
					
				}
                }
				}
				}
            }
            }
			
		post
              {
			  always{
                     script
                     {
							echo "Skipped mail"
							//emailusers = userEmailList()
							//echo emailusers
                             //mail(from: "ReleaseUpdates@solartis.com", 
                             //to: emailusers, 
							 //cc:rel_team_user,	
                            // subject: "Release-${customer}-${Release_Number}",
                            // body: "Release Success  in BETA for ticket  ${Release_Number} \n To enter RC Environment Wait for approvals.Following assignees involved to approve:\nadmin\nqauser\nPM Kindly check log in below link:\nFor blue ocean view: ${RUN_DISPLAY_URL}\nFor classic view: ${BUILD_URL}" )
						approverusers = approveuser_list("${LOB}_QA")
						env.RELEASE_SCOPE2 = input id: 'QAAppBeta', message: 'QA Approval Required', ok: 'Release!', submitter: approverusers,
                            parameters: [choice(name: 'RELEASE_SCOPE', choices: 'NO\nYES', description: 'Promote RC Release')]
						if (env.RELEASE_SCOPE2 == "NO") {
						approverusers = approveuser_list("ReleaseEngineering")
							env.RELEASE_TESTAPPROVAL = input id: 'RollBackBeta', message: 'Rollback Approval Required', ok: 'Done!', submitter: approverusers,
							parameters: [choice(name: 'TestVerificationApproval', choices: 'NO\nYES', description: 'Approve to Rollback')]
				
					    if (env.RELEASE_TESTAPPROVAL == "YES") {
							echo "Rollback"
							roll_back()
						}
						}
						approverusers = approveuser_list("${LOB}_DEV")
						env.RELEASE_SCOPE3 = input id: 'DevAppBeta', message: 'PM Approval Required', ok: 'Release!', submitter: approverusers,
                            parameters: [choice(name: 'RELEASE_SCOPE', choices: 'NO\nYES', description: 'Promote RC Release')]
						if (env.RELEASE_SCOPE3 == "NO") {
						approverusers = approveuser_list("ReleaseEngineering")
							env.RELEASE_TESTAPPROVAL = input id: 'RollBackBeta', message: 'Rollback Approval Required', ok: 'Done!', submitter: approverusers,
							parameters: [choice(name: 'TestVerificationApproval', choices: 'NO\nYES', description: 'Approve to Rollback')]
						if (env.RELEASE_TESTAPPROVAL == "YES") {
							echo "Rollback"
							roll_back()
						}
						}
					 }
                 }
                 
	     }

    }
	 	  stage('RC') {
		agent { node {  label 'ConfigTier'  } }
             steps {
script {
				   try{
				   count =0;
                   wrap([$class: 'BuildUser']) {
                   currentBuild.description = "${Release_Number}-${Support_Ticket} triggered by ${BUILD_USER}"
                   }
                stages_initial_Process()
				echo "SQL Execution and Backup..."
                sql_backup_execution()
				echo "Forms Backup and Deployment"
		     	form_deploy()
				echo "KB Backup and Deployment"
				kb_deploy()
				noderestart()
				approverusers=approveuser_list("ReleaseEngineering")
				env.RELEASE_TESTAPPROVAL = input id: 'RelRC', message: 'Release Team Approval Required', ok: 'Done!', submitter: approverusers,
                parameters: [choice(name: 'TestVerificationApproval', choices: 'NO\nYES', description: 'Release Verification Test Completed')]

                if (env.RELEASE_TESTAPPROVAL == "NO") {
					env.RELEASE_TESTAPPROVAL = input id: 'RollBackRC', message: 'Rollback Approval Required', ok: 'Done!', submitter: approverusers,
                parameters: [choice(name: 'TestVerificationApproval', choices: 'NO\nYES', description: 'Approve to Rollback')]
				
				if (env.RELEASE_TESTAPPROVAL == "YES") {
					echo "Rollback"
					//sh '/usr/bin/Solartis/CD_Automation/Rollback/rollbackv2.sh ${Support_Ticket}-${Release_Number}'
				}
                }
                }
				catch(Exception e)
				{
 				echo "${e}"
				retry(10){
				    echo "Skipped mail"
				//emailusers=userEmailList()
				if(count == 0){
				    echo "Skipped mail"
                //mail(from: "ReleaseUpdates@solartis.com", 
                         //    to: emailusers, 
						//	 cc: rel_team_user,	
                         //    subject: "Release-${customer}-${Release_Number}",
                         //    body: "Deployment Failed in RC for ticket  ${Release_Number} \n Fix the issue and click Rerun. Kindly check log in below link:\nFor blue ocean view: ${RUN_DISPLAY_URL}\nFor classic view: ${BUILD_URL}" )
                     			   
			} 
			else{
			    echo "Skipped mail"
		//	mail(from: "ReleaseUpdates@solartis.com", 
                  //           to: emailusers, 
					//		 cc: rel_team_user,
                     //        subject: "Release-${customer}-${Release_Number}",
                      //       body: "Deployment Again Failed in RC for ticket  ${Release_Number} \n Fix the issue and click Rerun Kindly check log in below link:\nFor blue ocean view: ${RUN_DISPLAY_URL}\nFor classic view: ${BUILD_URL}" )
			}
				approverusers=approveuser_list("${LOB}_DEV")
			env.RELEASE_SCOPE = input id: 'ReRunRC', message: 'Task execution  Failed in RC  Once issue Fixed Leave Comment and rerun', ok: 'RERUN!', submitter: approverusers,
                            parameters: [string(name: 'Aprroval_string', defaultvalue: '', description: 'Approval comment:')]  
			count =count+1;
				stages_initial_Process()
				echo "SQL Execution and Backup..."
				sql_backup_execution()
				echo "Forms Backup and Deployment"
		     	form_deploy() 
				echo "KB Backup and Deployment"
				kb_deploy()
				noderestart()
				approverusers = approveuser_list("ReleaseEngineering")
				
				env.RELEASE_TESTAPPROVAL = input id: 'RelRCReRun', message: 'Release Team Approval Required', ok: 'Done!', submitter: approverusers,
                parameters: [choice(name: 'TestVerificationApproval', choices: 'NO\nYES', description: 'Release Verification Test Completed')]

                if (env.RELEASE_TESTAPPROVAL == "NO") {
					env.RELEASE_TESTAPPROVAL = input id: 'RollBackRCReRun', message: 'Rollback Approval Required', ok: 'Done!', submitter: approverusers,
                parameters: [choice(name: 'TestVerificationApproval', choices: 'NO\nYES', description: 'Approve to Rollback')]
				
				if (env.RELEASE_TESTAPPROVAL == "YES") {
					echo "Rollback"
					roll_back()
					
				}
                }
				}
				}
            }
            }
			post
              {
                 always{
                     script
                     {
							echo "Skipped mail"
						//	emailusers = userEmailList()
							//echo emailusers
                            // mail(from: "ReleaseUpdates@solartis.com", 
                             //to: emailusers, 
							// cc:rel_team_user,	
                            // subject: "Release-${customer}-${Release_Number}",
                            // body: "Release Success  in RC for ticket  ${Release_Number} \n To enter GA Environment Wait for approvals.Following assignees involved to approve:\nadmin\nqauser\nPM Kindly check log in below link:\nFor blue ocean view: ${RUN_DISPLAY_URL}\nFor classic view: ${BUILD_URL}" )
						approverusers = approveuser_list("${LOB}_QA")
						env.RELEASE_SCOPE2 = input id: 'QAAppRC', message: 'QA Approval Required', ok: 'Release!', submitter: approverusers,
                            parameters: [choice(name: 'RELEASE_SCOPE', choices: 'NO\nYES', description: 'Promote GA Release')]
						if (env.RELEASE_SCOPE2 == "NO") {
						approverusers = approveuser_list("ReleaseEngineering")
							env.RELEASE_TESTAPPROVAL = input id: 'RollBackRCReRun', message: 'Rollback Approval Required', ok: 'Done!', submitter: approverusers,
							parameters: [choice(name: 'TestVerificationApproval', choices: 'NO\nYES', description: 'Approve to Rollback')]
				
					    if (env.RELEASE_TESTAPPROVAL == "YES") {
							echo "Rollback"
							roll_back()
						}
						}
						approverusers = approveuser_list("${LOB}_DEV")
						env.RELEASE_SCOPE3 = input id: 'DevAppRC', message: 'PM Approval Required', ok: 'Release!', submitter: approverusers,
                            parameters: [choice(name: 'RELEASE_SCOPE', choices: 'NO\nYES', description: 'Promote GA Release')]
						if (env.RELEASE_SCOPE3 == "NO") {
						approverusers = approveuser_list("ReleaseEngineering")
							env.RELEASE_TESTAPPROVAL = input id: 'RollBackRCDev', message: 'Rollback Approval Required', ok: 'Done!', submitter: approverusers,
							parameters: [choice(name: 'TestVerificationApproval', choices: 'NO\nYES', description: 'Approve to Rollback')]
						if (env.RELEASE_TESTAPPROVAL == "YES") {
							echo "Rollback"
							roll_back()
						}
						}
						approverusers = approveuser_list("BUSUSER")
						env.RELEASE_SCOPE3 = input id: 'BUSAppGA', message: 'BUS Approval Required', ok: 'Release!', submitter: approverusers,
                            parameters: [choice(name: 'RELEASE_SCOPE', choices: 'NO\nYES', description: 'Promote GA Release')]
						if (env.RELEASE_SCOPE3 == "NO") {
						approverusers = approveuser_list("ReleaseEngineering")
							env.RELEASE_TESTAPPROVAL = input id: 'RollBackRCBUS', message: 'Rollback Approval Required', ok: 'Done!', submitter: approverusers,
							parameters: [choice(name: 'TestVerificationApproval', choices: 'NO\nYES', description: 'Approve to Rollback')]
						if (env.RELEASE_TESTAPPROVAL == "YES") {
							echo "Rollback"
							roll_back()
						}
						}
					 }
                 }
	     }
    }
        stage('GA') {
		agent { node {  label 'ConfigTier'  } }
            steps {
                script {
				try{
				 count =0;
                   wrap([$class: 'BuildUser']) {
                   currentBuild.description = "${Release_Number}-${Support_Ticket} triggered by ${BUILD_USER}"
					}
                    current_time()			
					if (formattedDate >= '04:30:00 AM' || formattedDate <= '09:30:00 PM' ){
					approverusers = approveuser_list("INFRA")

					env.RELEASE_SCOPE = input message: 'Production Hours Approval for release', ok: 'Done!', submitter: approverusers,
                            parameters: [choice(name: 'Aprroval_choice', choices: 'NO\nYES', description: 'Approve GA Release')]
					}	
				stages_initial_Process()   
				echo "SQL Execution and Backup..."
				sql_backup_execution()
				echo "Forms Backup and Deployment"
		     	form_deploy() 
				echo "KB Backup and Deployment"
				kb_deploy()
				noderestart()
				approverusers = approveuser_list("ReleaseEngineering")
				env.RELEASE_TESTAPPROVAL = input message: 'Release Team Approval Required', ok: 'Done!', submitter: approverusers,
                parameters: [choice(name: 'TestVerificationApproval', choices: 'NO\nYES', description: 'Release Verification Test Completed')]

                if (env.RELEASE_TESTAPPROVAL == "NO") {
					env.RELEASE_TESTAPPROVAL = input message: 'Rollback Approval Required', ok: 'Done!', submitter: approverusers,
                parameters: [choice(name: 'TestVerificationApproval', choices: 'NO\nYES', description: 'Approve to Rollback')]
				
				if (env.RELEASE_TESTAPPROVAL == "YES") {
					echo "Rollback"
					//sh '/usr/bin/Solartis/CD_Automation/Rollback/rollbackv2.sh ${Support_Ticket}-${Release_Number}'
				}
                }
     }
              catch(Exception e)	{
			  echo "${e}"
			  retry(10){
			      echo "Skipped mail"
			  //emailusers=userEmailList()
			  count =count+1;
			  if(count == 0){
			      echo "Skipped mail"
               //mail(from: "ReleaseUpdates@solartis.com", 
                //             to:emailusers, 
				//			 cc:rel_team_user,
                    //         subject: "Release-${customer}-${Release_Number}",
                         //    body: "Deployment Failed in GA for ticket  ${Release_Number} \n Fix the issue and click Rerun \nPM Kindly check log in below link:\nFor blue ocean view: ${RUN_DISPLAY_URL}\nFor classic view: ${BUILD_URL}" )
                     			   
			} 
			else{
			    echo "Skipped mail"
		//	mail(from: "ReleaseUpdates@solartis.com", 
                  //           to: emailusers, 
				//			 cc:rel_team_user,	
                           //  subject: "Release-${customer}-${Release_Number}",
                           //  body: "Deployment Failed in GA for ticket  ${Release_Number} \n Fix the issue and click Rerun \nPM Kindly check log in below link:\nFor blue ocean view: ${RUN_DISPLAY_URL}\nFor classic view: ${BUILD_URL}" )
			}
			approverusers = approveuser_list("${LOB}_DEV")
			env.RELEASE_SCOPE = input message: 'Task execution  Failed in GA  Once issue Fixed Leave Comment and rerun', ok: 'RERUN!', submitter: approverusers,
            parameters: [string(name: 'Aprroval_string', defaultvalue: '', description: 'Approval comment:')]
			    stages_initial_Process()   
				echo "SQL Execution and Backup..."
				sql_backup_execution()
				echo "Forms Backup and Deployment"
		     	form_deploy() 
				echo "KB Backup and Deployment"
				kb_deploy()
				noderestart()
				approverusers = approveuser_list("ReleaseEngineering")
				env.RELEASE_TESTAPPROVAL = input message: 'Release Team Approval Required', ok: 'Done!', submitter: approverusers,
                parameters: [choice(name: 'TestVerificationApproval', choices: 'NO\nYES', description: 'Release Verification Test Completed')]

                if (env.RELEASE_TESTAPPROVAL == "NO") {
					env.RELEASE_TESTAPPROVAL = input message: 'Rollback Approval Required', ok: 'Done!', submitter: approverusers,
                parameters: [choice(name: 'TestVerificationApproval', choices: 'NO\nYES', description: 'Approve to Rollback')]
				if (env.RELEASE_TESTAPPROVAL == "YES") {
					echo "Rollback"
					//sh '/usr/bin/Solartis/CD_Automation/Rollback/rollbackv2.sh ${Support_Ticket}-${Release_Number}'
				}

                }
			  }
			  }			 
              }
            }
		 post
              {
                 always{
                     script
                     {
                         echo "Skipped mail"
					 //emailusers=userEmailList()
                       //      mail(from: "ReleaseUpdates@solartis.com", 
                        //     to: emailusers, 
						//	 cc:rel_team_user,
                            // subject: "Release-${customer}-${Release_Number}",
                           //  body: "Release Success  in Alpha,Beta,RC and GA for ticket  ${Release_Number} \n Release completed\n Kindly check log in below link:\nFor blue ocean view: ${RUN_DISPLAY_URL}\nFor classic view: ${BUILD_URL} " )
                 }
             }
	      }
		  }
    }
  }
