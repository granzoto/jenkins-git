pipeline {
    agent any
    stages {
        stage('--cleanup--') {
            steps {
                // Stop all ducker containers
                sh "docker stop $(docker ps -f "name=ducker*" --format="{{.Names}}") 2>/dev/null || echo 'No more containers to remove.'"
                
               // Remove all ducker containers
               sh "docker rm -vf \$(docker ps -f "name=ducker*" --format="{{.Names}}") 2>/dev/null || echo 'No more containers to remove.'"

               // Remove all ducker images
               sh "docker rmi \$(docker ps -f "name=ducker*" --format="{{.Names}}") -f 2>/dev/null || echo 'No more images to remove.'"
            }
        }
        stage('--checkout--') {
            steps {
                // Checkout kafka source from granzoto repo
                // TODO : Switch to real kafka repo

                checkout([$class: 'GitSCM', branches: [[name: '*/granzoto-test-jenkins']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/granzoto/kafka']]])
            }
        }
        stage('--build--') {
            steps {
                // Steps to setup the envirnoment and run the tests
                // Steps :
                //     1 - Setup pythin virtualenv and activate it
                //     2 - Install ducktape
                //     3 - Run tests
                // TODO : Check if we can make it modular

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
