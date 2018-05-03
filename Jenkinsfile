pipeline {
    agent any

    triggers {
        pollSCM('*/1 * * * *')
    }

    parameters {
        booleanParam(defaultValue: true, description: 'Switch if DEV deployment stage is active (and following stages)', name: 'DEPLOY_BUILD')
    }

    stages {
        stage('Output'){
            steps {
                sh 'echo "This tests if outputing works"'
            }
            steps {
                echo "Parameter value is: ${params.DEPLOY_BUILD}"
            }
        }
        stage('Test') {
            /* 
            when {
                expression { params.DEPLOY_BUILD }
            }
            */
            steps {
                sh 'echo "Does this message still get displayed?"'
            }
        }
    }
}