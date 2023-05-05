pipeline {
    agent any
    stages {
        stage('Initialize'){
            steps{
                echo "Esta es el inicio"
            }
        }
        stage('Sonar'){
            steps{
                sh '/var/jenkins_home/sonar-scanner/bin/sonar-scanner \
                -Dsonar.projectKey=NewSpringMvcRest \
                -Dsonar.java.binaries=. \
                -Dsonar.sources=. \
                -Dsonar.host.url=http://192.168.1.89:9001 \
                -Dsonar.login=squ_7dbd979576414da98dee7a51c762fe9d8c1091f8'
            }
        }
        stage('Build') {
            steps {
                sh 'mvn -B package'
            }
            post {
                success {
                    slackSend color: '#36a64f', message: 'Build succeeded!'
                }
                failure {
                    slackSend color: '#ff0000', message: 'Build failed :('
                }
                unstable {
                    slackSend color: '#f4b942', message: 'Build unstable :/'
                }
                aborted {
                    slackSend color: '#c9c9c9', message: 'Build aborted.'
                }
            }
        }
            
        stage('Test') {
            steps {
                sh "mvn clean verify" 
            }
            post {
                always {
                    slackSend color: '#439FE0', message: 'Tests complete.'
                }
            }
        } 
        stage("Publish to Nexus Repository Manager") {
            steps {
                script {
                    pom = readMavenPom file: "pom.xml";
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                    artifactPath = filesByGlob[0].path;
                    artifactExists = fileExists artifactPath;
                    if(artifactExists) {
                        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}";
                        nexusArtifactUploader(
                            nexusVersion: "nexus3",
                            protocol: "http",
                            nexusUrl: "192.168.1.89:8082",
                            groupId: pom.groupId,
                            version: pom.version,
                            repository: "NewSpringMvcRest-artefactos",
                            credentialsId: "df3e1d58-db3d-449f-bb6f-9a64000f3542",
                            artifacts: [
                                [artifactId: pom.artifactId,
                                        classifier: '',
                                        file: artifactPath,
                                        type: pom.packaging]
                            ]
                        );
                    } else {
                        error "*** File: ${artifactPath}, could not be found";
                    }
                }
            }
            post {
                success {
                    slackSend color: '#36a64f', message: 'Deployment to Nexus succeeded!'
                }
                failure {
                    slackSend color: '#ff0000', message: 'Deployment to Nexus failed :('
                }
            }
        }
    }
    post {
        success {
            slackSend color: '#36a64f', message: 'Job complete.'
        }
        failure {
            slackSend color: '#ff0000', message: 'Job failed :('
        }
    } 
}

def slackSend(def opts) {
    def color = opts.color ?: '#36a64f'
    def message = opts.message ?: ''
    def slackToken = env.SLACK_TOKEN ?: ''
    def slackChannel = env.SLACK_CHANNEL ?: ''
    if (slackToken == '') {
        echo 'No Slack token found.'
        return
    }
    if (slackChannel == '') {
        echo 'No Slack channel found.'
        return
    }
    sh """
        curl -X POST \
        --data-urlencode 'payload={
            \"channel\": \"${slackChannel}\",
            \"attachments\": [{
                \"color\": \"${color}\",
                \"text\": \"${message}\"
            }]
        }' \
        https://hooks.slack.com/services/${slackToken}
    """
}
