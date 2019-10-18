pipeline {
    agent none
    environment{
        KONG_PACKAGE_NAME = 'kong'
        REPOSITORY_NAME = 'kong-nightly'
        REPOSITORY_OS_NAME = "${env.BRANCH_NAME}"
        KONG_VERSION = "2019-10-17"
        UPDATE_CACHE = "true"
        DOCKER_CREDENTIALS = credentials('dockerhub')
        DOCKER_USERNAME = "${env.DOCKER_CREDENTIALS_USR}"
        DOCKER_PASSWORD = "${env.DOCKER_CREDENTIALS_PSW}"
        BINTRAY_CREDENTIALS = credentials('bintray-ce')
        BINTRAY_USR = "${env.BINTRAY_CREDENTIALS_USR}"
        BINTRAY_KEY = "${env.BINTRAY_CREDENTIALS_PSW}"
        AWS_ACCESS_KEY = credentials('AWS_ACCESS_KEY')
        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
        KONG_BUILD_TOOLS = "origin/feat/kong-jenkins"
        BUILDX = "false"
        DOCKER_MACHINE_ARM64_NAME = "jenkins-kong-${env.BUILD_NUMBER}"
    }
    stages {
        stage('Nightly Releases') {
            parallel {
                stage('Ubuntu Releases') {
                    agent {
                        node {
                            label 'docker-compose'
                        }
                    }
                    environment {
                        PACKAGE_TYPE = 'deb'
                        RESTY_IMAGE_BASE = 'ubuntu'
                        RESTY_IMAGE_TAG = 'trusty'
                        KONG_SOURCE_LOCATION = "${env.WORKSPACE}"
                        KONG_BUILD_TOOLS_LOCATION = "${env.WORKSPACE}/../kong-build-tools"
                    }
                    steps {
                        sh 'printenv'
                        sh 'make setup-kong-build-tools'
                        sh 'mkdir -p $HOME/bin'
                        sh 'sudo ln -s $HOME/bin/kubectl /usr/local/bin/kubectl'
                        sh 'sudo ln -s $HOME/bin/kind /usr/local/bin/kind'
                        dir('../kong-build-tools'){ sh 'make setup-ci' }
                        sh 'export RESTY_IMAGE_TAG=trusty && make nightly-release'
                        dir('../kong-build-tools'){ sh 'make setup-ci' }
                        sh 'export BUILDX=true RESTY_IMAGE_TAG=xenial && make nightly-release'
                        dir('../kong-build-tools'){ sh 'make setup-ci' }
                        sh 'export RESTY_IMAGE_TAG=bionic && make nightly-release'
                    }
                }
                stage('Debian Releases') {
                    agent {
                        node {
                            label 'docker-compose'
                        }
                    }
                    environment {
                        PACKAGE_TYPE = 'deb'
                        RESTY_IMAGE_BASE = 'debian'
                        KONG_SOURCE_LOCATION = "${env.WORKSPACE}"
                        KONG_BUILD_TOOLS_LOCATION = "${env.WORKSPACE}/../kong-build-tools"
                    }
                    steps {
                        sh 'printenv'
                        sh 'make setup-kong-build-tools'
                        sh 'mkdir -p $HOME/bin'
                        sh 'sudo ln -s $HOME/bin/kubectl /usr/local/bin/kubectl'
                        sh 'sudo ln -s $HOME/bin/kind /usr/local/bin/kind'
                        dir('../kong-build-tools'){ sh 'make setup-ci' }
                        sh 'export RESTY_IMAGE_TAG=jessie && make nightly-release'
                        dir('../kong-build-tools'){ sh 'make setup-ci' }
                        sh 'export RESTY_IMAGE_TAG=stretch && make nightly-release'
                    }
                }
            }
        }
    }
}