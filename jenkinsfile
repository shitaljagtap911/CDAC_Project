pipeline {
  agent any 
  stages {
    stage ('Initialize') {
      steps {
        sh '''
                echo "PATH = ${PATH}"
                echo "M2_HOME = ${M2_HOME}"
            ''' 
      }
    }
    
    stage ('Check secrets') {
      steps {
      sh 'trufflehog3 https://github.com/shitaljagtap911/CDAC_Project.git -f json -o truffelhog_output.json || true'
      }
    }
    
    stage ('Software composition analysis') {
            steps {
                dependencyCheck additionalArguments: ''' 
                    -o "./" 
                    -s "./"
                    -f "ALL" 
                    --prettyPrint''', odcInstallation: 'DP-Check'

                dependencyCheckPublisher pattern: 'dependency-check-report.xml'
            }
        }
    
    stage ('Static analysis') {
      steps {
        withSonarQubeEnv('sonarqube') {
          sh 'mvn sonar:sonar'
        //sh 'sudo python3 Devsecops.py'
	}
      }
    }

    stage ('Generate build') {
      steps {
        sh 'mvn clean install'
      }
    }

    stage ('Deploy to server') {
            steps {
           sshagent(['application_server']) {
                sh 'scp -o StrictHostKeyChecking=no /var/lib/jenkins/workspace/DemoProject/webgoat-server/target/webgoat-server-v8.2.0-SNAPSHOT.jar ubuntu@65.2.191.49:/WebGoat'
                sh 'ssh -o  StrictHostKeyChecking=no ubuntu@65.2.191.49 "nohup java -jar /WebGoat/webgoat-server-v8.2.0-SNAPSHOT.jar &"'
              } 
           }
    }

    stage ('Dynamic analysis') {
            steps {
           sshagent(['application_server']) {
                sh 'ssh -o  StrictHostKeyChecking=no ubuntu@13.233.216.101 "sudo docker run --rm -v /home/ubuntu:/zap/wrk/:rw -t owasp/zap2docker-stable zap-full-scan.py -t http://65.2.191.49/WebGoat -x zap_report || true" '
                sh 'ssh -o  StrictHostKeyChecking=no ubuntu@13.233.216.101 "sudo ./zap_report.sh"'
              }
           }
    }

   // stage ('Host vulnerability assessment') {
   //     steps {
  //           sh 'echo "In-Progress"'
   //         }
   // }

  // stage ('Security monitoring and misconfigurations') {
  //      steps {
 //             sh 'echo "AWS misconfiguration"'
            // sh './securityhub.sh'
   //         }
   // }


   stage ('Incidents report') {
        steps {
        sh 'echo "Final Report"'
         sh './final_report.sh'
        }
    }

   }
}
