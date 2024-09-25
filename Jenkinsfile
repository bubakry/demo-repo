pipeline {
    agent any

    tools {
        maven 'Maven3'
    }
    stages {
        stage ('Build') {
            steps {
                sh 'mvn clean install -f MyWebApp/pom.xml'
            }
        }
        stage ('Code Quality') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn -f MyWebApp/pom.xml sonar:sonar'
                }
            }
        }
        stage ('JaCoCo') {
            steps {
                jacoco(
                    execPattern: 'MyWebApp/target/jacoco.exec',
                    classPattern: 'MyWebApp/target/classes',
                    sourcePattern: 'MyWebApp/src/main/java',
                    inclusionPattern: '**/*.class',
                    exclusionPattern: '**/*Test*.class'
                )
            }
        }
        stage ('Nexus Upload') {
            steps {
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: 'ec2-18-205-34-5.compute-1.amazonaws.com:8081',
                    groupId: 'com.dept.app',
                    version: '1.0-SNAPSHOT',
                    repository: 'maven-snapshots',
                    credentialsId: 'a91ea448-5887-4d9e-bc96-3ee9d43a6b25',
                    artifacts: [
                        [artifactId: 'MyWebApp',
                         classifier: '',
                         file: 'MyWebApp/target/MyWebApp.war',
                         type: 'war']
                    ]
                )
            }
        }
        stage ('DEV Deploy') {
            steps {
                echo "deploying to DEV Env"
                deploy adapters: [tomcat9(credentialsId: 'Tomcat credentials', path: '', url: 'http://ec2-98-82-39-195.compute-1.amazonaws.com:8080')], contextPath: null, war: '**/*.war'
            }
        }
        stage ('Slack Notification') {
            steps {
                echo "deployed to DEV Env successfully"
                slackSend(channel: 'devops-demo-project', message: "Deployment to Dev Env was successful! Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
            }
        }
        stage ('DEV Approve') {
            steps {
                echo "Taking approval from DEV Manager for QA Deployment"
                timeout(time: 7, unit: 'DAYS') {
                    input message: 'Do you want to deploy?', submitter: 'admin'
                }
            }
        }
        stage ('QA Deploy') {
            steps {
                echo "deploying to QA Env"
                deploy adapters: [tomcat9(credentialsId: 'Tomcat credentials', path: '', url: 'http://ec2-98-82-39-195.compute-1.amazonaws.com:8080')], contextPath: null, war: '**/*.war'
            }
        }
        stage ('QA Approve') {
            steps {
                echo "Taking approval from QA manager"
                timeout(time: 7, unit: 'DAYS') {
                    input message: 'Do you want to proceed to PROD?', submitter: 'admin,manager_userid'
                }
            }
        }
        stage ('Slack Notification for QA Deploy') {
            steps {
                echo "deployed to QA Env successfully"
                slackSend(channel: 'devops-demo-project', message: "Job is successful! Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
            }
        }
    }
}
