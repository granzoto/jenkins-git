pipeline {
    agent any
    parameters {
        string(name: 'duckerUID',
          defaultValue: '1000',
          description: 'Ducker UID')
        string(name: 'duckerGID',
          defaultValue: '1000',
          description: 'Ducker GroupID')
    }
    stages {
        stage('--cleanup--') {
            steps {
                 // Debug
                 sh "docker ps -a -f \"name=ducker*\" --format=\"{{.Names}}\""

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
                // Checkout kafka source from granzoto repo
                // TODO : Switch to real kafka repo

                // RG checkout([$class: 'GitSCM', branches: [[name: '*/granzoto-test-jenkins']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/granzoto/kafka']]])
                checkout([$class: 'GitSCM', branches: [[name: '*/trunk']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/apache/kafka']]])
            }
        }
        stage('--prepare--') {
            steps {
                // Remove ARG from begining
                sh "sed -i -e \"/^ARG jdk_version/d\" tests/docker/DockerFile"

               // Set JDK Version
               // TODO : Make this a parameter
               sh "sed -i -e \"s/^FROM \\$jdk_version/FROM openjdk:8/g\" tests/docker/DockerFile"

               // Check which Kafka and sttrream version do we need to test.
               // --- DockerFile	2019-05-01 13:38:52.221039934 +0200
               // +++ DockerFile-Jakub	2019-05-01 13:16:52.883400949 +0200
               // -RUN mkdir -p "/opt/kafka-2.0.1" && chmod a+rw /opt/kafka-2.0.1 && curl -s "$KAFKA_MIRROR/kafka_2.12-2.0.1.tgz" | tar xz --strip-components=1 -C "/opt/kafka-2.0.1"
               // -RUN mkdir -p "/opt/kafka-2.1.0" && chmod a+rw /opt/kafka-2.1.0 && curl -s "$KAFKA_MIRROR/kafka_2.12-2.1.0.tgz" | tar xz --strip-components=1 -C "/opt/kafka-2.1.0"
               // +RUN mkdir -p "/opt/kafka-2.0.0" && chmod a+rw /opt/kafka-2.0.0 && curl -s "$KAFKA_MIRROR/kafka_2.12-2.0.0.tgz" | tar xz --strip-components=1 -C "/opt/kafka-2.0.0"

               // Add user duker
               sh "echo \"UID ${params.duckerUID}\""
               sh "echo \"GID ${params.duckerGID}\""
            }
        }
        stage('--build--') {
            steps {
                // Steps to setup the environment and run the tests
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
