pipeline {
   agent any

   tools {
      // Install the Maven version configured as "M3" and add it to the path.
      maven "maven"
   }

   stages {
   	stage ('SCM'){
   		steps {
   			git 'https://github.com/vikash21kumar/WebApp.git'
   		}
   	}
   	stage ('Compile and Static Code analysis'){
   		steps {
   			
   			sh "mvn sonar:sonar -Dsonar.host.url=http://40.112.52.164:9000/ -Dsonar.login=admin -Dsonar.password=admin  -Dsonar.sources=. -Dsonar.tests=. -Dsonar.test.inclusions=**/test/java/servlet/createpage_junit.java -Dsonar.exclusions=**/test/java/servlet/createpage_junit.java"
   		}
   	}
      stage('Build') {
         steps {
            // Get some code from a GitHub repository
            

            // Run Maven on a Unix agent.
            sh "mvn -Dmaven.test.failure.ignore=true clean package"

            // To run Maven on a Windows agent, use
            // bat "mvn -Dmaven.test.failure.ignore=true clean package"
         }
        
         post {
            // If Maven was able to run the tests, even if some of the test
            // failed, record the test results and archive the jar file.
            success {
               junit '**/target/surefire-reports/TEST-*.xml'
               archiveArtifacts 'target/*.war'
            }
         }
      }
   }
}
