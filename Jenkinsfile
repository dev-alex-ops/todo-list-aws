pipeline {
    agent any
    
    stages {
        stage('Get Code') {
            steps{
                git url: 'https://github.com/dev-alex-ops/todo-list-aws'
                sh 'git checkout develop'
                sh 'wget https://raw.githubusercontent.com/dev-alex-ops/todo-list-aws-config/staging/samconfig.toml'
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
                    export API_ID=$(aws apigateway get-rest-apis --query items[0].id --output text)
                    export BASE_URL="https://$API_ID.execute-api.us-east-1.amazonaws.com/Prod"
                    python -m pytest --junitxml=junit-rest.xml test/integration/todoApiTest.py
                '''
                junit 'junit-rest.xml'
            }
        }
        
        stage('Promote') {
            steps{
                withCredentials([string(credentialsId: 'fa29894d-bf6a-49d6-97a7-5d30f879e8fb', variable: 'gh_token')]) {
                    sh 'rm junit-rest.xml flake8.out bandit.out'
                    sh 'git checkout master'
                    sh 'git pull'
                    
                    catchError(buildResult: 'SUCCESS', stageResult: 'SUCCESS') {
                        sh 'git merge --no-commit develop'
                    }
                    
                    sh '''
                        git checkout master -- Jenkinsfile
                        git add Jenkinsfile
                        git commit -am "Merge branch 'develop' into 'master' while keeping Jenkinsfile from master"
                        git push https://dev-alex-ops:$gh_token@github.com/dev-alex-ops/todo-list-aws.git
                    '''
                }
            }
        }
        
        stage ('Clean Workspace') {
            steps {
                cleanWs deleteDirs: true, patterns: [[pattern: '**', type: 'INCLUDE']]
            }
        }
    }
}