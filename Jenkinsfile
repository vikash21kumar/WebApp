pipeline {
    agent any
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

                rtMavenRun (
                    tool: "maven", // Tool name from Jenkins configuration
                    pom: 'pom.xml',
                    goals: 'clean install',
                    deployerId: "MAVEN_DEPLOYER",
                    resolverId: "MAVEN_RESOLVER"
                )
            }
            post {
                always {
                jiraSendBuildInfo branch: 'DEMO-1', site: 'txdevopsbootcamp.atlassian.net'
                jiraComment body: 'Build ${BUILD_NUMBER} completed with commit ${GIT_COMMIT}', issueKey: 'DEMO-1'
                
                //jiraSendBuildInfo branch: 'master', site: 'txdevopsbootcamp.atlassian.net'
       }
   }
        }
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
       }
        }
        }
        

        stage ('Publish build info') {
            steps {
                rtPublishBuildInfo (
                    serverId: "txdevops"
                )
            }
        }
    }
}
