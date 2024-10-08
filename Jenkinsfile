pipeline {
    agent any
    options {
        skipDefaultCheckout(true)
    }
    stages {
        stage('Code checkout from GitHub') {
            steps {
                script {
                    cleanWs()
                    git credentialsId: 'gh-abcd-student', url: 'https://github.com/tastycookie/abcd-student', branch: 'main'
                }
            }
        }

    
        stage('Example') {
            steps {
                echo 'Hello!'
                sh 'ls -la'
            }
        }
    }
}
