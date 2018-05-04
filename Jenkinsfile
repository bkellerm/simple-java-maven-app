pipeline {
    agent any
    
    triggers {
        //pollSCM '@hourly'
        pollSCM 'H/5 * * * *'
    }

    parameters {
        string(defaultValue: 'master', description: 'Branch name to filter the DEV deployment', name: 'BRANCH_FILTER')
        booleanParam(defaultValue: true, description: 'Switch if DEV deployment stage is active (and following stages)', name: 'DEPLOY_BUILD')
    }
    
    tools {
        maven 'maven339'
        jdk 'jdk8'
    }
    options {
        // Keep the 5 most recent builds
        buildDiscarder(logRotator(numToKeepStr:'5'))
    }
    stages {
        stage ('Initialize') {
            steps {
                step([$class: 'WsCleanup'])
                checkout scm   //need to install SCM plugin "pipeline scm step plugin?"
                sh '''
                    echo "PATH = ${PATH}"
                    echo "M2_HOME = ${M2_HOME}"
                '''
            }
        }
        stage ('Build') {
            // use the local docker host for building the container image
            environment {
                DOCKER_HOST = 'unix:///var/run/docker.sock'
                USE_SSH_FOR_GIT_REPOSITORIES = 1
            }
            steps {
                sh 'mvn help:system'
                // Change the snapshot version in all pom.xml files to a release candidate version
                setVersionSuffix()
                versiontiger('.')
                // Build with Maven (the tmp directory must be changed from /tmp)
                sh 'mvn -B -Djava.io.tmpdir=${WORKSPACE} clean install -Porganisation_inventage -Pstyleguide'     //changed to organisation inventage
                // read the actual version from pom and save it as env variable
                setVersion() // as env.VERSION
            }        
        }
        stage ('Smoke Test') {
            steps {
                sh '''
                    echo 'Smoke Test'
                '''
                    // start SUT via docker-compose
                    // docker run --rm -e SUT=http://com.vpbank.portal.server:18080 --network=docker_portal-net nexus.vpbank.net:5000/com.vpbank.portal.acceptance-tests:$VERSION specs/pre-deploy/Healthcheck.spec
            }        
        }
         stage ('DEV: Deployment') {
            when {
                branch params.BRANCH_FILTER
                expression { params.DEPLOY_BUILD } // true := automatisches Deployment auf DEV    //run when brach matches name and deploy_build true
            }
            steps {
                sh '''
                    ssh docker@llidockcport01e.vpbank.net 'docker ps'

                    echo 'Copy configuration' \
                    && scp compose/target/classes/portal.common.env docker@llidockcport01e.vpbank.net:/opt/vpb/docker/ && scp compose/target/classes/portal.DEV.env docker@llidockcport01e.vpbank.net:/opt/vpb/docker/portal.specific.env && scp compose/target/classes/docker-compose.yml docker@llidockcport01e.vpbank.net:/opt/vpb/docker/

                    echo 'Clean old container' \
                    && ssh docker@llidockcport01e.vpbank.net 'sudo docker-compose -f /opt/vpb/docker/docker-compose.yml down && sudo docker-compose -f /opt/vpb/docker/docker-compose.yml rm -f'

                    echo 'Deploy new container' \
                    && ssh docker@llidockcport01e.vpbank.net 'sudo docker-compose -f /opt/vpb/docker/docker-compose.yml up -d'

                    ssh docker@llidockcport01e.vpbank.net 'docker ps'
                '''
            }
        }
        stage ('DEV: Acceptance Test') {
            when {
                branch params.BRANCH_FILTER
                expression { false } // true := Aktivierung der Akzeptanztestausführung
            }
            steps {
                sh '''
                    echo 'Login Test'
                    ssh docker@llicichk001.vpbank.net "docker run --rm --add-host www.vpbank-dev.com:80.241.113.10 -e SUT=https://www.vpbank-dev.com nexus.vpbank.net:5000/com.vpbank.portal.acceptance-tests:$VERSION specs/post-deploy/website"
                '''
                //    ssh docker@llicichk001.vpbank.net "docker run --rm --add-host www.vpbank-dev.com:80.241.113.10 -e SUT=https://www.vpbank-dev.com nexus.vpbank.net:5000/com.vpbank.portal.acceptance-tests:$VERSION specs/post-deploy/ebanking"
                //    ssh docker@llicichk001.vpbank.net "docker run --rm --add-host www.vpbank-dev.com:80.241.113.10 -e SUT=https://www.vpbank-dev.com nexus.vpbank.net:5000/com.vpbank.portal.acceptance-tests:$VERSION specs/post-deploy/prolink"
                //    ssh docker@llicichk001.vpbank.net "docker run --rm --add-host www.vpbank-dev.com:80.241.113.10 -e SUT=https://www.vpbank-dev.com nexus.vpbank.net:5000/com.vpbank.portal.acceptance-tests:$VERSION specs/post-deploy/iam"
            }
        }

        /* We don't want to deploy
        stage ('Store Deployment Config in Repo') {
            when {
                expression { params.DEPLOY_BUILD }
            }
            steps {
                sh '''
                    echo 'Store Deployment Config com.vpbank.portal.docker-compose in Repo'
                    echo "Upload docker-compose:$VERSION.jar to Nexus repository" \
                    && curl -k -v --netrc-file ~/.netrc --noproxy '*' --upload-file compose/target/docker-compose-$VERSION.jar https://nexus.vpbank.net:8443/repository/vpb-artifact-next/com.vpbank.portal.docker-compose-$VERSION.jar
                '''
            }
        }
        stage ('Trigger Deployment Pipeline') {
            when {
                branch params.BRANCH_FILTER
                expression { params.DEPLOY_BUILD }
            }
            steps {
                build job: 'NEXT/Portal/Portal-Deployment', parameters: [string(name: 'PACKAGE_NAME', value: "com.vpbank.portal"), string(name: 'PACKAGE_VERSION', value: "${env.VERSION}")]
            }
        }
        */
    }
    post {
        changed {
            //emailext body: '${DEFAULT_CONTENT}', subject: '${DEFAULT_SUBJECT}', to: '$DEFAULT_RECIPIENTS'
            sh 'echo "POST: changed"'
        }
        unstable {
            /*
            emailext body: '${DEFAULT_CONTENT}', subject: '${DEFAULT_SUBJECT}', to: '$DEFAULT_RECIPIENTS'
            script {
                currentBuild.result = 'UNSTABLE'
            }
            */
            sh 'echo "POST: unstable"'
        }
        failure {
            /*
            emailext body: '${DEFAULT_CONTENT}', subject: '${DEFAULT_SUBJECT}', to: '$DEFAULT_RECIPIENTS'
            script {
                currentBuild.result = 'FAILURE'
            }
            */
            sh 'echo "POST: failure"'
        }
    }

}

/** 
 * Sets the environment variable 'VERSION_SUFFIX' to yyyyMMddhhmm-${BUILD_NUMBER}-GitCommitHash
 */
def setVersionSuffix() {
    def now = sh (script: 'date -u +%Y%m%d%H%M', returnStdout: true).trim()
    def gitCommitHash = sh (script: 'git rev-parse --short HEAD', returnStdout: true).trim()
    env.VERSION_SUFFIX=now +"-"+ "${env.BUILD_NUMBER}" +"-"+ gitCommitHash
}

def versiontiger(reactorFolder) {
    dir(reactorFolder) {
        sh 'mvn com.inventage.tools.versiontiger:versiontiger-maven-plugin:execute -B -DstatementsFile=jenkins.versiontiger'
    }
}

def setVersion() {
    def pom = readMavenPom file: 'pom.xml'
    env.VERSION = pom.version
}
