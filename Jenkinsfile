pipeline {
    agent any
    
    stages {
        stage('Get Code') {
            steps{
                git url: 'https://github.com/dev-alex-ops/todo-list-aws'
                sh 'wget https://raw.githubusercontent.com/dev-alex-ops/todo-list-aws-config/production/samconfig.toml'
            }
        }

        stage('Deploy') {
            steps{
                sh '''
                    sam build
                    sam validate --region eu-east-1
                    sam deploy --config-file samconfig.toml --config-env production --no-fail-on-empty-changeset
                '''
            }
        }
    
        stage('Rest Test') {
            steps{
                sh '''
                    export API_ID=$(aws apigateway get-rest-apis --query items[0].id --output text)
                    export BASE_URL="https://$API_ID.execute-api.us-east-1.amazonaws.com/Prod"
                    python -m pytest -m read --junitxml=junit-rest.xml test/integration/todoApiTest.py
                '''
                junit 'junit-rest.xml'
            }
        }
        
        
        stage ('Clean Workspace') {
            steps {
                cleanWs deleteDirs: true, patterns: [[pattern: '**', type: 'INCLUDE']]
            }
        }
    }
}