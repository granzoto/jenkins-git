pipeline {
    agent any
    parameters {
        string(name: 'duckerUID',
          defaultValue: '1000',
          description: 'Ducker UID')
        string(name: 'duckerGID',
          defaultValue: '1000',
          description: 'Ducker GroupID')
        string(name: 'testPath',
          defaultValue: 'tests/kafkatest/tests/client/pluggable_test.py',
          description: 'Test path definition')
        string(name: 'JDK_VERSION',
          defaultValue: 'openjdk:8',
          description: 'JDK Version')
    }
    stages {
        stage('--cleanup--') {
            steps {
                // Stop all ducker containers
                sh "docker stop \$(docker ps -a -f \"name=ducker*\" --format=\"{{.Names}}\") 2>/dev/null || echo 'No more containers to remove.'"
                
               // Remove all ducker containers
               sh "docker rm -vf \$(docker ps -a -f \"name=ducker*\" --format=\"{{.Names}}\") 2>/dev/null || echo 'No more containers to remove.'"

               // Remove all ducker images
               sh "docker rmi \$(docker ps -a -f \"name=ducker*\" --format=\"{{.Names}}\") -f 2>/dev/null || echo 'No more images to remove.'"
            }
        }
        stage('--checkout--') {
            steps {
                // Checkout kafka source from real Kafka repo

                // RG checkout([$class: 'GitSCM', branches: [[name: '*/granzoto-test-jenkins']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/granzoto/kafka']]])
                checkout([$class: 'GitSCM', branches: [[name: '*/trunk']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/apache/kafka']]])
            }
        }
        stage('--prepare--') {
            steps {
                // Remove ARG from begining
                sh "sed -i -e \"/^ARG jdk_version/d\" tests/docker/Dockerfile"

               // Set JDK Version in Dockerfile
               sh "sed -i -e \"s/^FROM .jdk_version/FROM ${params.JDK_VERSION}/g\" tests/docker/Dockerfile"

               // Set TimeZone in Dockerfile
               sh "sed -i -e \"s#America/Los_Angeles#Europe/Prague#g\" tests/docker/Dockerfile"

               // Add user duker with UID and GID parameters
               sh "sed -i -e \"s/^RUN useradd -ms/RUN groupadd -g ${params.duckerGID} ducker \\&\\& useradd -g ${params.duckerGID} -u ${params.duckerUID} -ms/g\" tests/docker/Dockerfile"
            }
        }
        stage('--build--') {
            steps {
                // Steps to setup the environment and run the tests
                // Steps :
                //     1 - Setup python virtualenv and activate it
                //     2 - Install ducktape
                //     3 - Run tests
                // TODO : Check if we can make it modular

                sh "echo \"Test for parameter substitution - ${params.testPath} - The value should be inside the hifens\""

                sh '''
                    echo "Setting up virtualEnv"
                    virtualenv streams-rhel
                    . ./streams-rhel/bin/activate
                    echo "Install ducktape"
                    pip install ducktape
                    echo "Run a single test"
                '''  
                sh "echo \"Run a single test\""
                sh ". ./streams-rhel/bin/activate && TC_PATHS=${params.testPath} tests/docker/run_tests.sh"
            }
        }
    }
}
