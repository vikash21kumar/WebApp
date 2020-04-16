pipeline {
    agent any
    tools {
      // Install the Maven version configured as "M3" and add it to the path.
      maven "maven"
   }
    stages {
        stage ('Clone') {
            steps {
                git branch: 'DEMO-1', url: "https://github.com/vikash21kumar/WebApp.git"
                //sh Commit_message=`git log --format="medium" -1 ${GIT_COMMIT}`# print commit, author, date, title & commit message
            }
        }

        stage ('Artifactory configuration') {
            steps {
                rtServer (
                    id: "txdevops",
                    url: "https://txdevopsbootcamp.jfrog.io/txdevopsbootcamp",
                    credentialsId: "artifactory-txdevops"
                )

                rtMavenDeployer (
                    id: "MAVEN_DEPLOYER",
                    serverId: "txdevops",
                    releaseRepo: "libs-release-local",
                    snapshotRepo: "libs-snapshot-local"
                )

                rtMavenResolver (
                    id: "MAVEN_RESOLVER",
                    serverId: "txdevops",
                    releaseRepo: "libs-release",
                    snapshotRepo: "libs-snapshot"
                )
                
            }
        }

        stage ('Exec Maven') {
            steps {
                slackSend channel: '#cicd', message: 'Build Started'
                rtMavenRun (
                                tool: "maven", // Tool name from Jenkins configuration
                                pom: 'pom.xml',
                                goals: 'clean install ',
                                deployerId: "MAVEN_DEPLOYER",
                                resolverId: "MAVEN_RESOLVER"
                            )
                    }
            post {
                always {
                    slackSend channel: '#cicd', message: 'Build Completed '
                    jiraSendBuildInfo branch: 'DEMO-1', site: 'txdevopsbootcamp.atlassian.net'
                    jiraComment body: "Build 'env.BUILD_NUMBER' completed with commit ", issueKey: 'DEMO-1'
                    //comment_issues()
                    //jiraSendBuildInfo branch: 'master', site: 'txdevopsbootcamp.atlassian.net'
                    }
                }
        }
        stage ('SonarQube analysis') { 
                steps{

                withSonarQubeEnv('sonarqube') { 
               
                sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.3.0.603:sonar ' + 
                '-f pom.xml ' +
                '-Dsonar.projectKey=JavaWebApp ' +
                '-Dsonar.login=admin ' +
                '-Dsonar.password=admin ' +
                '-Dsonar.language=java ' +
                '-Dsonar.sources=. ' +
                '-Dsonar.tests=. ' +
                '-Dsonar.test.inclusions=**/test/java/servlet/createpage_junit.java ' +
                '-Dsonar.exclusions=**/test/java/servlet/createpage_junit.java'
                }

                }
                

        }
        //stage ("SonarQube Quality Gate") { 
        //        steps {
         //           script{

         //           timeout(time: 2, unit: 'MINUTES') { 
         //       def qg = waitForQualityGate() 
         //       if (qg.status != 'OK') {
         //       error "Pipeline aborted due to quality gate failure: ${qg.status}"
         //       }
         //           }
                    

         //       }
                
         //   }
        //}
        stage ('List Artifact') {
            steps {
                sh "ls -ltr ${WORKSPACE}/target/JavaWebApp-1.0.0.101.war"
                sh "ls -ltr /var/lib/jenkins/keys/caseStudy.pem"
                sh "chmod 400  /var/lib/jenkins/keys/caseStudy.pem"
                //scp -i /path/my-key-pair.pem /path/SampleFile.txt ec2-user@ec2-198-51-100-1.compute-1.amazonaws.com:~
            }
        }
        stage ('Functional Test') {
            steps {
                rtMavenRun (
                    tool: "maven", // Tool name from Jenkins configuration
                    pom: 'functionaltest/pom.xml',
                    goals: 'test',
                    deployerId: "MAVEN_DEPLOYER",
                   resolverId: "MAVEN_RESOLVER"
                )
            }
            
           post {
                success {
                  // publish html
                  publishHTML target: [
                      allowMissing: false,
                     alwaysLinkToLastBuild: false,
                      keepAll: true,
                      reportDir: 'functionaltest/target/surefire-reports',
                     reportFiles: 'index.html',
                     reportName: 'Functional Test Report'
                    ]
                slackSend channel: '#cicd', message: 'Functional Test Completed '
                }
            
            }
       }
        stage ('QA Deployment') {
           
            steps {
                //sh "ls -ltr ${WORKSPACE}/target/JavaWebApp-1.0.0.101.war"
                sh "scp -i /var/lib/jenkins/keys/caseStudy.pem  ${WORKSPACE}/target/JavaWebApp-1.0.0.101.war ubuntu@3.19.222.141:"
                sh "ssh -i /var/lib/jenkins/keys/caseStudy.pem  ubuntu@3.19.222.141 sudo mv JavaWebApp-1.0.0.101.war QAWebapp.war"
                sh "ssh -i /var/lib/jenkins/keys/caseStudy.pem  ubuntu@3.19.222.141 sudo cp *.war /opt/tomcat/webapps/"
                //sh "ssh -i /var/lib/jenkins/keys/caseStudy.pem  ubuntu@3.19.222.141 sudo chown tomcat:tomcat /opt/tomcat/webapps/*.war"

            }
        post {
                always {
                jiraSendDeploymentInfo environmentId: 'QA', environmentName: 'QA', environmentType: 'testing', serviceIds: ['DEMO-1'], site: 'txdevopsbootcamp.atlassian.net', state: 'successful'
                slackSend channel: '#cicd', message: 'QA Deployment  Completed '
       }
        }
        }
        
    stage ('BlazeMeter test'){
             steps {
                 blazeMeterTest credentialsId:'Blazemeter',
                 serverUrl:'https://a.blazemeter.com',
                 testId:'7883232',
                 notes:'',
                 sessionProperties:'',
                 jtlPath:'',
                 junitPath:'',
                 getJtl:false,
                 getJunit:false
                 }
             }

        stage ('Publish build info') {
            steps {
                rtPublishBuildInfo (
                    serverId: "txdevops"
                )
            }
        }

        stage ('PROD Deployment') {
           
            steps {
                sh "ls -ltr ${WORKSPACE}/target/JavaWebApp-1.0.0.101.war"
                sh "scp -i /var/lib/jenkins/keys/caseStudy.pem  ${WORKSPACE}/target/JavaWebApp-1.0.0.101.war ubuntu@13.58.247.254:"
                sh "ssh -i /var/lib/jenkins/keys/caseStudy.pem  ubuntu@13.58.247.254 sudo mv JavaWebApp-1.0.0.101.war ProdWebapp.war"
                sh "ssh -i /var/lib/jenkins/keys/caseStudy.pem  ubuntu@13.58.247.254 sudo cp *.war /opt/tomcat/webapps/"
                //sh "ssh -i /var/lib/jenkins/keys/caseStudy.pem  ubuntu@18.223.162.120 sudo chown tomcat:tomcat /opt/tomcat/webapps/*.war"

            }
        post {
                always {
                jiraSendDeploymentInfo environmentId: 'PROD', environmentName: 'PROD', environmentType: 'production', serviceIds: ['DEMO-1'], site: 'txdevopsbootcamp.atlassian.net', state: 'successful'
                slackSend channel: '#cicd', message: 'Production Deployment Completed '
       }
        }
        }

        stage ('Acceptance Test') {
            steps {
                rtMavenRun (
                    tool: "maven", // Tool name from Jenkins configuration
                    pom: 'Acceptancetest/pom.xml',
                    goals: 'test',
                    deployerId: "MAVEN_DEPLOYER",
                    resolverId: "MAVEN_RESOLVER"
                )
            }
            
            post {
                success {
                  // publish html
                  publishHTML target: [
                      allowMissing: false,
                      alwaysLinkToLastBuild: false,
                      keepAll: true,
                      reportDir: 'Acceptancetest/target/surefire-reports',
                      reportFiles: 'index.html',
                      reportName: 'Acceptance Test Report'
                    ]
                slackSend channel: '#cicd', message: 'Acceptace Test Completed '
                }
            }
        }

    }
}

void comment_issues() {
    def issue_pattern = "DEMO-\\d"

    // Find all relevant commit ids
    currentBuild.changeSets.each {changeSet ->
        changeSet.each { commit ->
            String msg = commit.getMsg()
            msg.findAll(issue_pattern).each {
                // Actually post a comment
                id -> jiraComment idOrKey: 'DEMO', comment: 'Hi there!'
            }
        }
    }
}
