pipeline {
    agent any
    
    environment {
        AWS_REGION = 'us-east-1' 
        STACK_NAME = 'todo-list-aws' 
        SAM_CONFIG_FILE = 'samconfig.toml' 
        SAM_CONFIG_ENV = 'staging'
        PATH = "/var/lib/jenkins/.local/bin:${env.PATH}"
    }

    stages {
        stage('Get Code') {
            steps {
                script {
                   sh '''
                        whoami
                        hostname
                        echo ${WORKSPACE}
                    '''
                    checkout([$class: 'GitSCM', branches: [[name: '*/develop']], userRemoteConfigs: [[url: 'https://github.com/JGilPantoja/todolist-aws.git']]])
                    sh '''
                        ls -la
                    '''
                }
            }
        }

        stage('Static Test') {
            parallel {
                stage('Flake8') {
                    steps {
                        script {
                            sh '''
                                whoami
                                hostname
                                echo ${WORKSPACE}
                                flake8 src --exit-zero --format=pylint > flake8_report.txt || true
                            '''
                            archiveArtifacts artifacts: 'flake8_report.txt', fingerprint: true
                        }
                    }
                }

                stage('Bandit') {
                    steps {
                        script {
                            sh '''
                                whoami
                                hostname
                                echo ${WORKSPACE}
                                bandit -r src -f txt -o bandit_report.txt || true
                            '''
                            archiveArtifacts artifacts: 'bandit_report.txt', fingerprint: true
                        }
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    sh '''
                        whoami
                        hostname
                        echo ${WORKSPACE}
                        
                        sam build --config-file $SAM_CONFIG_FILE --config-env $SAM_CONFIG_ENV --region $AWS_REGION
        
                        sam validate --config-file $SAM_CONFIG_FILE --config-env $SAM_CONFIG_ENV --region $AWS_REGION
                        
                        sam deploy --config-file $SAM_CONFIG_FILE \
                                   --config-env $SAM_CONFIG_ENV \
                                   --region $AWS_REGION \
                                   --no-confirm-changeset \
                                   --capabilities CAPABILITY_IAM
                    '''
                }
            }
        }

    }
}
