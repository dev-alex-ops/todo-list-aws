pipeline {
    agent any
    
    stages {
        stage('Get Code') {
            steps{
                git url: 'https://github.com/dev-alex-ops/todo-list-aws', branch: 'develop'
            }
        }

        stage('Static Test') {
            steps{
                sh '''
                    python -m flake8 --format=pylint --exit-zero src > flake8.out
                    python -m bandit --exit-zero -r src -f custom -o bandit.out --severity-level medium --msg-template "{abspath}:{line}: [{test_id}] {msg}"
                '''
                recordIssues tools: [flake8(pattern: 'flake8.out')]
                recordIssues tools: [pyLint(name: 'Bandit', pattern: 'bandit.out')]
            }
        }

        stage('SAM') {
            steps{
                sh '''
                    sam build
                    sam validate --region eu-east-1
                    sam deploy --config-file samconfig.toml --config-env staging --no-fail-on-empty-changeset
                '''
            }
        }
    
        stage('Rest Test') {
            steps{
                sh '''
                    export API_ID=$(aws apigateway get-rest-apis --query items[?contains(name, 'staging')].id --output textS
                    export BASE_URL="https://$API_ID.execute-api.us-east-1.amazonaws.com/Prod"
                    python -m pytest test/integration/todoApiTest.py
                '''
            }
        }
        
        stage('Promote') {
            steps{
                sh '''
                    git tag -a RC -m "Release version"
                    git merge develop
                    git commit -am "Merged develop branch to master"
                    git push origin master
                '''
            }
        }
    }
}