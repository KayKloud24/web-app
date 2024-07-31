COLOR_MAP = [
    'SUCCESS': 'good',
    'FAILURE': 'danger',
]

pipeline {
    agent any
    tools {
        maven "maven3.9.8"
    }

    stages {
        stage ("Git clone") {
            steps {
                git branch: 'main', url: 'https://github.com/KayKloud24/web-app.git'
            }
        }
        
        stage ("Build with maven") {
            steps {
                sh "mvn clean"
            }
        }

        stage ("Testing with maven") {
            steps {
                sh "mvn test"
            }
        }

        stage ("Packaging with maven") {
            steps {
                sh "mvn package"
            }
        }

        stage('SonarQube Analysis') {
            environment {
                ScannerHome = tool 'sonar5.0'
            }
            steps {
                script {
                    withSonarQubeEnv('sonar') {
                        sh "${ScannerHome}/bin/sonar-scanner -Dsonar.projectKey=kloud"
                    }
                }
            }
        }

        stage("Quality Gate") {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage("Upload to Nexus") {
            steps {
                nexusArtifactUploader artifacts: [[artifactId: 'maven-web-application', classifier: '', file: '/var/lib/jenkins/workspace/First-Pipeline-Job/target/web-app.war', type: 'war']], credentialsId: 'nexus-id', groupId: 'com.mt', nexusUrl: '98.81.18.43:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'webapp-release', version: '3.8.1-RELEASE'
            }
        }

        stage("Deploy to UAT") {
            steps {
                deploy adapters: [tomcat9(credentialsId: 'tomcred', path: '', url: 'http://3.236.231.155:8080/')], contextPath: null, war: 'target/*.war'
            }
        }

    }

    post {
        always {
            slackSend channel: 'team6-africa', 
                      color: COLOR_MAP[currentBuild.currentResult],
                      message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER}"
        }
    }

}