build_number = "${env.BUILD_NUMBER}"
job = env.JOB_NAME.split('/')
job_name = job[0]
branch_name = job[1]
git_branch_name = branch_name.replaceAll("%2F","/")
url_branch_name = git_branch_name.replaceAll("/","%252F")

sf config set org-api-loglevel=debug

node {
       try
        {
		stage('Checkout') {
		checkout scm
		}


		script {
                    // Reading a username and password credential
                    withCredentials([usernamePassword(credentialsId: 'my-cred-id', 
                                                     usernameVariable: 'USERNAME', 
                                                     passwordVariable: 'PASSWORD')]) {
                        echo "Using Username: ${USERNAME}"
                        // You can use the password variable, but never echo it to keep it secure
                        sh '''
                        echo "Executing some command with $USERNAME"
                        '''
                    }

                    // Reading a secret text credential
                    withCredentials([string(credentialsId: 'my-secret-text-id', variable: 'SECRET_TEXT')]) {
                        echo "Secret Text: ${SECRET_TEXT}" // For demonstration purposes only; avoid echoing secrets in production.
                    }

                    // Reading a secret file credential
                    withCredentials([file(credentialsId: 'my-secret-file-id', variable: 'SECRET_FILE')]) {
                        echo "Using Secure File at: ${SECRET_FILE}"
                        sh '''
                        cat $SECRET_FILE # Example usage; avoid exposing file contents in logs
                        '''
                    }
                
		stage('Build_Backend_Code') {
		echo "Running: Build_Backend_Code"
		sh '''set +x
		'''
		}

	        stage('UT_Code') {
                echo "Running : UT_Backend_Code"
		}
}  
                catch(e)
        {
		stage('Email_Notification_For_Failures') {
			notifyBuild("FAILED","${e}")
			step([$class: 'WsCleanup', cleanWhenFailure: true])
		  		}
		throw e
	}
			notifyBuild(currentBuild.result,"NULL")
}

def sendMail(String buildStat,String errr) {
	def subject = "${buildStat}: Job '${job} [${build_number}]'"
	def summary = "${subject}\nError was: ${errr}"
	sh "git log --after 1.days.ago|egrep -io '[a-z0-9\\-\\._@]++\\.[a-z0-9]{1,4}'|head -1 >lastAuthor"
	sh "set +x;sed -i 's/ //g' lastAuthor"	//fixing the email adddress
  		def lines = readFile("lastAuthor")
  		println "Email notifications will be send to : ${lines}"
  		//use try catch to work with email validation issues
try {
	println "\u2713 ${lines} is valid email"
        mail bcc: '', body: "${summary}", charset: 'UTF-8', mimeType: 'text/plain', subject: "${subject}", to: "mayuri.patil@reancloud.com", from: 'mayuri.patil@reancloud.com'	
} catch(err) {
	def errsubject = "Error Sending Mail: ${buildStat}, Job '${job} [${build_number}]'"
	def errsummary = "${errsubject}\nError was: ${err}.\n${lines} is not a valid mail address.\nPlease check the reason why the mail is failing.\n\nThis is an auto-generated mail, please do not reply back.\nif you find any descrepancies in the mail content, kindly write to  Monali Reddy or Shrivallabh Deshmukh with proper description."
	def errPeople = "mayuri.patil@reancloud.com"
	println "\u274C ${lines} is not valid email"
	mail bcc: '', body: "${errsummary}", charset: 'UTF-8', mimeType: 'text/plain', subject: "${errsubject}", to: "${errPeople}", from: 'mayuri.patil@reancloud.com'
}
}

def notifyBuild(String buildStatus = 'STARTED',String thiserr) {
  buildStatus =  buildStatus ?: 'SUCCESSFUL'
	
	def colorName = 'RED'
  	def colorCode = '#FF0000'
	
  if (buildStatus == 'STARTED') {
    color = 'YELLOW'
    colorCode = '#FFFF00'
  } else if (buildStatus == 'SUCCESSFUL') {
    color = 'GREEN'
    colorCode = '#00FF00'
	  step([$class: 'GitHubCommitStatusSetter',
        contextSource: [$class: 'ManuallyEnteredCommitContextSource',
        context: 'BUILD STATUS'],
        statusResultSource: [$class: 'ConditionalStatusResultSource',
        results: [[$class: 'AnyBuildResult',
        message: 'SUCCESSFUL',
        state: currentBuild.result]]]])
        echo "status set to ${buildStatus}."
	  sendMail("SUCCESSFUL","NULL")
  } else if (buildStatus == 'FAILED') {
    color = 'RED'
    colorCode = '#FF0000'
	  step([$class: 'GitHubCommitStatusSetter',
        contextSource: [$class: 'ManuallyEnteredCommitContextSource',
        context: 'BUILD STATUS'],
        statusResultSource: [$class: 'ConditionalStatusResultSource',
        results: [[$class: 'AnyBuildResult',
        message: 'FAILED',
        state: '${buildStatus}']]]])
        echo "status set to ${buildStatus}."
	  sendMail("FAILED","${thiserr}")
} 
	step([$class: 'WsCleanup', cleanWhenFailure: true])
}
