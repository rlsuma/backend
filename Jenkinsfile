pipeline {
    agent {
        label 'AGENT-2'
    }
    options {
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds()
        ansiColor('xterm')
    }
     environment{
        def appVersion = '' //variable declaration
        nexusUrl = 'nexus.rlsu:8081'
    }
    stages {
        stage('read the version'){
            steps{
                script{
                    def packageJson = readJSON file: 'package.json'
                    appVersion = packageJson.version
                    echo "application version: $appVersion"
                }
            }
        }
        stage('install dependencies') {
            steps {
               sh """
               npm install
               ls -ltr
               """
            }
        }
         stage('Build'){
            steps{
                sh """
                zip -q -r backend-${appVersion}.zip * -x Jenkinsfile -x backend-${appVersion}.zip
                ls -ltr
                """
            }
        }
      stage('Nexus Artifact Upload'){
            steps{
                script{
                    nexusArtifactUploader(
                          nexusVersion: 'nexus3',
                            protocol: 'http',
                            nexusUrl: 'http://nexus.rlsu:8081',
                            groupId: 'com.expense',
                            artifactId: 'backend',
                            version: '1.2.0',
                            repository: 'backend',
                            credentialsId: 'nexus-credentials', // Ensure this ID is correct
                                artifacts: [
                                    [artifactId: 'backend',
                                    classifier: '',
                                    file: 'backend-1.2.0.zip',
                                    type: 'zip']
                                ]
                            )
                }
            }
        } 

        stage ('Deploy') 
        steps {
            script{
                def params =[
                    string(name:'appVersion', value: "${appVersion}")
                ]
            build job: 'backend-deploy', parameters: params, wait: false
               }
        }

    }
    post {
        always {
            echo 'I will always say Hello again!'
            deleteDir()
        }
        success {
            echo 'I will run when pipeline is success'
        }
        failure {
            echo 'I will run when pipeline is failure'
        }
    }
}
