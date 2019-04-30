pipeline {
    agent any
    stages {
        stage('------checkout--------') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/granzoto-test-jenkins']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/granzoto/kafka']]])
            }
        }
        stage('------build--------') {
            steps {
                sh 'echo "Hello World"'
                sh '''
                    echo "Multiline shell steps works too"
                    ls -lah
                '''

                sh '''
                    echo "Setting up virtualEnv"
                    virtualenv streams-rhel
                    . ./streams-rhel/bin/activate
                    echo "Install ducktape"
                    pip install ducktape
                    echo "Run a single test"
                    TC_PATHS=tests/kafkatest/tests/client/pluggable_test.py tests/docker/run_tests.sh
                '''  
            }
        }
    }
}
