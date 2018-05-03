pipeline {
    agent { any
    /*
        docker {
            image 'maven:3-alpine'
            args '-v /root/.m2:/root/.m2'
        }
        */
    }

    parameters {
        //string(defaultValue: 'master', description: 'Branch name to filter the DEV deployment', name: 'BRANCH_FILTER')
        booleanParam(defaultValue: true, description: 'Switch if DEV deployment stage is active (and following stages)', name: 'DEPLOY_BUILD')
    }

    stages {
        /*
        stage('Build') {
            steps {
                sh 'mvn -B -DskipTests clean package'
            }
        }
        */
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