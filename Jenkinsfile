pipeline {
    options {
        skipDefaultCheckout()
        timestamps()
    }
    parameters {
        string(name: 'BUILD_VERSION', defaultValue: '', description: 'The build version to deploy (optional)')
    }
    agent {
        label 'internal-build.ncats'
    }
    triggers {
        pollSCM('H/5 * * * *')
    }
    environment {
        PROJECT_NAME = "labshare-kong"
        TYPE = "service"
        DOCKER_REPO_NAME = "684150170045.dkr.ecr.us-east-1.amazonaws.com/facility-api"
    }
    stages {
        stage('Clean') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build Version') {
            steps {
                script {
                    BUILD_VERSION_GENERATED = VersionNumber(
                        versionNumberString: 'v${BUILD_YEAR, XX}.${BUILD_MONTH, XX}${BUILD_DAY, XX}.${BUILDS_TODAY}',
                        projectStartDate:    '1970-01-01',
                        skipFailedBuilds:    true
                    )
                    currentBuild.displayName = BUILD_VERSION_GENERATED
                    env.BUILD_VERSION = BUILD_VERSION_GENERATED
                }
            }
        }
        stage('Deploy - CI') {
            agent {
                label 'aws-ci-docker-host-linux-1.ncats'
            }
            steps {
                fileOperations {
                    fileCopyOperation(includes: 'docker-compose.yml', targetLocation: 'docker-compose.yml'){
                        withAWS(credentials:'aws-jenkins-build') {
                            sh '''
                            export DOCKER_LOGIN="`aws ecr get-login --no-include-email --region us-east-1`"
                            $DOCKER_LOGIN
                            '''
                            ecrLogin()
                            script {
                                sh 'docker-compose down -v --rmi all | xargs echo'
                                sh 'docker-compose up -d'
                                sh 'docker start nginx-gen | xargs echo'
                                sh 'sleep 25s'
                            }
                        }
                    }
                }
            }
        }
    }
}
