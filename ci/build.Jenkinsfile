#!/usr/bin/env groovy

def gitShortCommit=''
def date=''
def image_name='milvus-operator'
def github='https://github.com/sharding-db/milvus-operator.git'
pipeline {
   options{
    disableConcurrentBuilds(abortPrevious: true)
    skipDefaultCheckout()
   }
   agent {
        kubernetes {
            label 'infra-api-ci'
            inheritFrom 'default'
            defaultContainer 'main'
            yamlFile 'ci/pod/ci.yaml'
            customWorkspace '/home/jenkins/agent/workspace'
        }
   }
    parameters {
        gitParameter branchFilter: 'origin/(.*)', defaultValue: 'master', name: 'BRANCH', type: 'PT_BRANCH'
    }
   environment{
      DOCKER_IMAGE="harbor-us-vdc.zilliz.cc/infra/${image_name}"
      GITHUB_TOKEN_ID="github-token"
      TEST_ENVIRONMENT="sit"
      ARGOCD_TOKEN_ID="argocd-token"
   }
    stages {
        stage('Checkout'){
            steps{
                git  credentialsId: 'zilliz-ci',  branch: "${params.BRANCH}", url: "${github}"
                script {
                    sh 'printenv'
                    date = sh(returnStdout: true, script: 'date +%Y%m%d').trim()
                    gitShortCommit = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
                    image_tag="${params.BRANCH}-${date}-${gitShortCommit}"
                }
            }
        }
        stage('Build & Publish Image') {
            steps{
                container(name: 'kaniko',shell: '/busybox/sh') {
                  script { 
                    sh 'ls -lah '
                    // Build & Publish Docker Image use kaniko
                    withCredentials([usernamePassword(credentialsId: "${env.GITHUB_TOKEN_ID}", usernameVariable: 'GITHUB_USER', passwordVariable: 'GITHUB_TOKEN')]){
                    sh """
                    executor \
                    --context="`pwd`" \
                    --registry-mirror="nexus-nexus-repository-manager-docker-5000.nexus:5000"\
                    --insecure-registry="nexus-nexus-repository-manager-docker-5000.nexus:5000" \
                    --build-arg=GITHUB_USER=${GITHUB_USER} \
                    --build-arg=GITHUB_TOKEN=${GITHUB_TOKEN} \
                    --dockerfile "Dockerfile" \
                    --destination=${DOCKER_IMAGE}:${image_tag}
                    """
                    }
                  }
                }
            }
        }
    }
}
