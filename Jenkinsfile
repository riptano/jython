#!/usr/bin/env groovy

@Library('ds-pipeline-lib')

import java.security.MessageDigest
def id = MessageDigest.getInstance("MD5").digest(System.currentTimeMillis().toString().bytes).encodeHex().toString().substring(0,8)

// Build Jython forked repository and push artifacts to Artifactory
// Addresses OPSC-17995: upgrades commons-compress 1.10 -> 1.27.1 to fix CVEs

pipeline {
    agent {
        node {
            label 'default-runner'
            customWorkspace "workspace/${BUILD_NUMBER}-package-build-${id}"
        }
    }

    tools {
        jdk('jdk-8')
    }

    parameters {
        string(name: 'buildbranch', defaultValue: 'ripcord-master', description: 'The branch to build.')
    }

    options {
        disableConcurrentBuilds()
        buildDiscarder(logRotator(numToKeepStr: '30'))
        timeout(time: 120, unit: 'MINUTES')
        timestamps()
    }

    stages {
        stage('Build') {
            steps {
                script {
                    env.JYTHON_VERSION = sh(
                        script: "grep -m1 'property name=\"jython.version\"' build.xml | sed 's/.*value=\"\\([^\"]*\\)\".*/\\1/'",
                        returnStdout: true
                    ).trim()
                    echo "Building jython-standalone version: ${env.JYTHON_VERSION}"
                }
                withAnt(installation: 'ant-1.10.7') {
                    sh "ant jar-standalone"
                    sh "mkdir -p artifacts"
                    sh "cp dist/jython-standalone-*.jar artifacts/"
                }
                archiveArtifacts artifacts: 'dist/jython-standalone-*.jar', onlyIfSuccessful: true, defaultExcludes: false, caseSensitive: false
            }
        }
        stage('Upload to Artifactory') {
            steps {
                script {
                    def directoryPath = 'artifacts'
                    def filenames = sh(script: "ls ${directoryPath}", returnStdout: true).trim().split('\n')

                    withCredentials([usernamePassword(credentialsId: 'dse-artifactory',
                                                      usernameVariable: 'ARTIFACTORY_USER',
                                                      passwordVariable: 'ARTIFACTORY_PASSWORD')]) {
                        for (def filename in filenames) {
                            sh "curl -sSf -u '${ARTIFACTORY_USER}:${ARTIFACTORY_PASSWORD}' -X PUT -T artifacts/${filename} 'https://repo.aws.dsinternal.org/artifactory/datastax-public-releases-local/com/datastax/opscenter/jython-standalone/${env.JYTHON_VERSION}/${filename}'"
                        }
                    }
                }
            }
        }
        stage('Wrapup') {
            steps {
                cleanWs notFailBuild: true
            }
        }
    }
}
