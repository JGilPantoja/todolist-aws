pipeline {
    agent any
    
    environment {
        AWS_REGION = 'us-east-1' 
        STACK_NAME = 'todo-list-aws-production' 
        SAM_CONFIG_FILE = 'samconfig.toml' 
        SAM_CONFIG_ENV = 'production'
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
                    checkout([$class: 'GitSCM', branches: [[name: '*/master']], userRemoteConfigs: [[url: 'https://github.com/JGilPantoja/todolist-aws.git']]])
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
                                   --capabilities CAPABILITY_IAM || \
                                   if [ $? -eq 1 ]; then echo "No changes to deploy, continuing..."; else exit 1; fi
                    '''
                }
            }
        }


        stage('Rest Test') {
            steps {
                script {
                    sh '''
                        whoami
                        hostname
                        echo ${WORKSPACE}
                        
                        export BASE_URL=$(aws cloudformation describe-stacks \
                            --stack-name $STACK_NAME --query "Stacks[0].Outputs[?OutputKey=='BaseUrlApi'].OutputValue" --output text)
        
                        echo "Base URL: $BASE_URL"
        
                        pytest -m read_only --maxfail=1 --disable-warnings test/integration/todoApiTest.py
                    '''
                }
            }
        }
    }
}
