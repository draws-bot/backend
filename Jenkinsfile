pipeline {
    agent {
        label 'AGENT-1'
    }
    options {
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds()
        ansiColor('xterm')
    }
    parameters{
        booleanParam(name: 'deploy', defaultValue: false, description: 'Toggle this value')
    }
    environment{
        def appVersion = '' //variable declaration
        nexusUrl = '54.85.71.212:8081'
        region = "us-east-1"
        account_id = "650732254329"
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
        stage('Install Dependencies') {
            steps {
               sh """
                npm install
                ls -ltr
                echo "application version: $appVersion"
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

        
        /* stage('Sonar Scan'){
            environment {
                scannerHome = tool 'sonar-6.0' //referring scanner CLI
            }
            steps {
                script {
                    withSonarQubeEnv('sonar-6.0') { //referring sonar server at manage-jenkins/systems 
                        sh "${scannerHome}/bin/sonar-scanner"
                    }
                }
            }
        } */

        stage('Nexus Artifact Upload'){
            steps{
                script{
                    nexusArtifactUploader(
                        nexusVersion: 'nexus3',
                        protocol: 'http',
                        nexusUrl: "${nexusUrl}",
                        groupId: 'com.expense',
                        version: "${appVersion}",
                        repository: "backend",
                        credentialsId: 'nexus-auth',
                        artifacts: [
                            [artifactId: "backend" ,
                            classifier: '',
                            file: "backend-" + "${appVersion}" + '.zip',
                            type: 'zip']
                        ]
                    )
                }
            }
        }
        stage('Deploy') {
            steps {
                sh 'echo this is deploy'
            }
        }
        /* stage('Deploy'){
            when{
                expression{
                    params.deploy
                }
            }
            steps{
                script{
                    def params = [
                        string(name: 'appVersion', value: "${appVersion}")
                    ]
                    build job: 'backend-deploy', parameters: params, wait: false
                }
            }
        } */
    }
    post { 
        always { 
            echo 'I will always say Hello again!'
            //deleteDir()
        }
        success { 
            echo 'I will run when pipeline is success'
        }
        failure { 
            echo 'I will run when pipeline is failure'
        }
    }
}