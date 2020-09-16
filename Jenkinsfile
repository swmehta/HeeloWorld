pipeline {
    agent {
        label "QA Environment"
    }
	environment {
        EMAIL_RECIPIENTS = 'swapmehta09@gmail.com'
    }
    stages {
        stage ('Initialize') {
            steps {
                bat '''
                    echo "JAVA_HOME = %JAVA_HOME%"
                    echo "M2_HOME = %M2_HOME%"
                '''
            }
        }

        stage('clean') {
            steps {
				bat 'mvn clean'
			}
        }
        stage("Unit testing") { 
			steps {
				script {
					bat 'mvn test' 
					junit '**//*target/surefire-reports/TEST-*.xml' 
                    archive 'target*//*.jar'
				}        
			}			
		}
        stage('package') {
            steps {
				bat 'mvn package'
            }
        }
		stage('artifact') {
			steps {
				archive 'target/*.war'
            }
			
		}
		stage('Sonar scan execution') {
            steps {
                bat "mvn verify sonar:sonar -Dintegration-tests.skip=true -Dmaven.test.failure.ignore=true"
            }
        }
		stage ('deploy'){
			steps {
				bat '''copy C:\\Users\\sbande\\Desktop\\201\\Jenkins\\slaves\\workspace\\EventRegistrationSystem_Pipeline\\target\\EventRegistrationSystem.war C:\\Users\\sbande\\Desktop\\201\\Jenkins\\Tomcat8\\webapps\\'''
            }
		}
	}
	post {
        success {
            sendEmail("Successful");
        }
        unstable {
            sendEmail("Unstable");
        }
        failure {
            sendEmail("Failed");
        }
    }
}

def developmentArtifactVersion = ''
def releasedVersion = ''
// get change log to be send over the mail
@NonCPS
def getChangeString() {
    MAX_MSG_LEN = 100
    def changeString = ""

    echo "Gathering SCM changes"
    def changeLogSets = currentBuild.changeSets
    for (int i = 0; i < changeLogSets.size(); i++) {
        def entries = changeLogSets[i].items
        for (int j = 0; j < entries.length; j++) {
            def entry = entries[j]
            truncated_msg = entry.msg.take(MAX_MSG_LEN)
            changeString += " - ${truncated_msg} [${entry.author}]\n"
        }
    }

    if (!changeString) {
        changeString = " - No new changes"
    }
    return changeString
}

def sendEmail(status) {
    mail(
            to: "$EMAIL_RECIPIENTS",
            subject: "Build $BUILD_NUMBER - " + status + " (${currentBuild.fullDisplayName})",
            body: "Changes:\n " + getChangeString() + "\n\n Check console output at: $BUILD_URL/console" + "\n")
}
