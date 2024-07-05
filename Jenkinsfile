#!/usr/bin/env groovy

@Library('ds-pipeline-lib')

import java.security.MessageDigest
def id = MessageDigest.getInstance("MD5").digest(System.currentTimeMillis().toString().bytes).encodeHex().toString().substring(0,8)

// Build Jython forked repository and push artifacts to artifactory

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
        string(name: 'buildbranch', defaultValue: 'OPSC-16690', description: 'The branch to build. Only valid for the branch build job.')
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
                    configFileProvider([configFile(fileId: 'gradle.properties',
                                                   replaceTokens: true,
                                                   targetLocation: 'gradle.properties')]) {
                        withAnt(installation: 'ant-1.10.7') {
                            sh "ant installer"
                            sh "mkdir artifacts"
                            sh "cp dist/jython-standalone* artifacts/"
                            sh "cp dist/jython-installer* artifacts/"
                        }
                    }
                archiveArtifacts artifacts: 'dist/jython-standalone-*.jar', onlyIfSuccessful: true, defaultExcludes: false, caseSensitive: false
                archiveArtifacts artifacts: 'dist/jython-installer-*.jar', onlyIfSuccessful: true, defaultExcludes: false, caseSensitive: false
            }
        }
        stage ('Upload to artifactory') {
            steps {
                script {
                    def directoryPath = 'artifacts'
                    def filenames = sh(script: "ls ${directoryPath}", returnStdout: true).trim().split('\n')

                    withCredentials([usernamePassword(credentialsId: 'dse-artifactory',
                                                  usernameVariable: 'ARTIFACTORY_USER',
                                                  passwordVariable: 'ARTIFACTORY_PASSWORD')]) {
                        for (def filename in filenames) {
                            sh "echo $filename"
                            sh "curl -sSf -u '$ARTIFACTORY_USER:$ARTIFACTORY_PASSWORD' -X PUT -T artifacts/$filename 'https://repo.aws.dsinternal.org/artifactory/datastax-public-releases-local/com/datastax/opscenter/jython-standalone/2.7.3a1/$filename'"
                        }
                    }
                }
            }
        }
        stage('wrapup') {
            steps {
                cleanWs notFailBuild: true
            }
        }
    }
}
