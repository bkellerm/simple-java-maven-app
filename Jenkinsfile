pipeline {
    agent any

    parameters {
        booleanParam(defaultValue: false, description: 'Switch if DEV deployment stage is active (and following stages)', name: 'DEPLOY_BUILD')
    }

    stages {
        stage('Output'){
            steps {
                sh 'echo "This tests if outputing works"'
            }
        }
        stage('Test') { 
            when {
                expression { params.DEPLOY_BUILD }
            }
            steps {
                sh 'echo "Does this message still get displayed?"'
            }
        }
    }
}