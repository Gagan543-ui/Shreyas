pipeline {
    agent any
   
    environment {
        GIT_URL = "https://github.com/opstree/spring3hibernate.git"
    }
   
    parameters {
        string(name: 'BRANCH', defaultValue: 'master', description: 'add branch name')
        choice(name: 'ENV', choices: ['Dev', 'Stage', 'Prod'], description: 'Pick something')
        booleanParam(name: 'SKIP_TESTS', defaultValue: false, description: 'skip test?')
    }
 
    stages {
        stage('Checkout') {
            steps {
                git branch: "${params.BRANCH}", url: "${env.GIT_URL}"
            }
        }
        stage('Compile') {
            steps {
                sh 'mvn compile'
            }
        }
        stage('parallel stage') {
            parallel {
                stage('GitLeaks') {
                    steps {
                        sh '''
                            echo "Running GitLeaks for secret scanning..."
                            gitleaks detect --source . --report-path=gitleaks-report.json || true
                        '''
                        archiveArtifacts artifacts: 'gitleaks-report.json', fingerprint: true, allowEmptyArchive: true
                    }
                }
                stage('Testing') {
                    when {
                       expression { return !params.SKIP_TESTS    }
                    }
                    steps {
                        sh 'mvn test'
                    }
                }
            }
       }
       stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }
        stage('Create Tag (Demo)') {
            steps {
                script {
                    def tagName = "spring3hibernate-v${env.BUILD_NUMBER}"
                    echo "Creating local tag for ${env.GIT_URL} → ${tagName}"
                    sh """
                        git fetch --tags
                        git tag ${tagName}
                        git show-ref --tags | grep ${tagName} || echo "Tag ${tagName} created locally"
                    """
                }
            }
        }
   
    }
    post {
        always {
            echo "Pipeline finished (success or failure)"
        }
        success {
            echo "Build succeeded!"
        }
        failure {
            echo "Build failed!"
        }
    }
}
 
   
 
