pipeline {
    agent any
    stages {
        stage('------checkout--------') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/granzoto-test-jenkins']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/granzoto/kafka']]])
            }
        }
        stage('Build') {
            steps {
                sh 'echo "Hello World"'
                sh '''
                    echo "Multiline shell steps works too"
                    ls -lah
                '''
            }
        }
    }
}
