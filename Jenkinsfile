pipeline {
    agent any
    
    environment {
        AWS_REGION = 'us-east-1' 
        STACK_NAME = 'todo-list-aws' 
        SAM_CONFIG_FILE = 'samconfig.toml' 
        SAM_CONFIG_ENV = 'default'
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
                            recordIssues(
                                tools: [flake8(name: 'Flake8', pattern: 'flake8_report.txt')]
                            )
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
                                bandit -r src -f custom -o bandit_report.txt --msg-template "{abspath}:{line}: [{test_id}] {msg}" || true
                            '''
                            recordIssues(
                                tools: [pyLint(name: 'Bandit', pattern: 'bandit_report.txt')]
                            )
                        }
                    }
                }
            }
            post {
                always {
                    archiveArtifacts artifacts: '*.txt', fingerprint: true
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
                        sam build --config-file $SAM_CONFIG_FILE --config-env $SAM_CONFIG_ENV
                        sam validate --config-file $SAM_CONFIG_FILE --config-env $SAM_CONFIG_ENV
                        sam deploy --stack-name $STACK_NAME \
                                   --region $AWS_REGION \
                                   --no-confirm-changeset \
                                   --no-fail-on-empty-changeset \
                                   --disable-rollback \
                                   --capabilities CAPABILITY_IAM \
                                   --parameter-overrides \
                                       Stage=default \
                                       AllowUnauthenticated=true
                    '''
                }
            }
        }
    }
}
