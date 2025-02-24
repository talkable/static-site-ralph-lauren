#!/usr/bin/env groovy

pipeline {
    agent {
        label "staging"
    }
    environment {
        GITHUB_ACC = credentials('talkable-bot-deploy-token')
        JENKINS_BUILD_USER_ID = ""
        SLACK_CHANNEL = "deploy"
        SLACK_CONTENT1 = "<${RUN_DISPLAY_URL}|${JOB_BASE_NAME}>\nResult: "
        SLACK_CONTENT2 = "\nExecuted on: *${NODE_NAME}*"
    }
    triggers {
        pollSCM('* * * * *') // Polls SCM every minute
    }
    options {
        buildDiscarder(logRotator(numToKeepStr: '5'))
        disableConcurrentBuilds()
        parallelsAlwaysFailFast()
        ansiColor('xterm')
    }
    stages {
        stage('Preparation') {
            steps {
                script {
                    sh 'printenv | sort'
                    JENKINS_BUILD_CAUSE = "${currentBuild.buildCauses.shortDescription}"
                    wrap([$class: 'BuildUser']) {
                        JENKINS_BUILD_USER_ID = "${BUILD_USER_ID}"
                        currentBuild.displayName = "${JOB_BASE_NAME}-${BUILD_NUMBER}"
                        currentBuild.description = "Build by ${JENKINS_BUILD_USER_ID}"
                    }
                    if (env.JOB_NAME == 'static.site.babbel') {
                        env.SITENAME = 'static-site-babbel'
                    } else if (env.JOB_NAME == 'static.site.leaf.home') {
                        env.SITENAME = 'static-site-leaf-home'
                    } else if (env.JOB_NAME == 'static.site.ralph.lauren') {
                        env.SITENAME = 'static-site-ralph-lauren'
                    } else if (env.JOB_NAME == 'static.site.rodanandfields') {
                        env.SITENAME = 'static-site-rodanandfields'
                    } else if (env.JOB_NAME == 'static.talkable.com') {
                        env.SITENAME = 'static-talkable-com'
                    } else {
                        echo "Hello from ${JOB_NAME} project! We don't build the ${JOB_NAME} right now. Please ask administrator to setup a job."
                        error("Non-supported project for build")
                    }
                }
            }
        }
        stage('Deploy') {
            steps([$class: 'BapSshPromotionPublisherPlugin']) {
                sshPublisher(continueOnError: false, failOnError: true,
                    publishers: [sshPublisherDesc(configName: "namecheapserver",
                        verbose: true,
                        transfers: [sshTransfer(execCommand: "cd talkable/${SITENAME}/ && git checkout main && git pull origin"),])])
            }
        }
    }
    post {
        success {
            echo 'Build was a success'
            slackSend(channel: "${SLACK_CHANNEL}", color: "#156e2a", message: "${SLACK_CONTENT1} *${currentBuild.result}!* ${SLACK_CONTENT2}\n${JENKINS_BUILD_CAUSE}")
        }
        failure {
            echo 'Build failed'
            slackSend(channel: "${SLACK_CHANNEL}", color: "#f00101", message: "${SLACK_CONTENT1} *${currentBuild.result}!* ${SLACK_CONTENT2}\n${JENKINS_BUILD_CAUSE}")
        }
        unstable {
            echo 'Build is unstable'
            slackSend(channel: "${SLACK_CHANNEL}", color: '#f5f63a', message: "${SLACK_CONTENT1} *${currentBuild.result}!* ${SLACK_CONTENT2}\n${JENKINS_BUILD_CAUSE}")
        }
        aborted {
            echo 'Build was aborted'
            slackSend(channel: "${SLACK_CHANNEL}", color: '#f5f63a', message: "${SLACK_CONTENT1} *${currentBuild.result}!* ${SLACK_CONTENT2}\n${JENKINS_BUILD_CAUSE}")
        }
        cleanup {
            echo 'Performing cleanup'
            cleanWs cleanWhenFailure: false,
                    cleanWhenUnstable: true,
                    cleanWhenAborted: true,
                    cleanWhenNotBuilt: true,
                    cleanWhenSuccess: true,
                    deleteDirs: true,
                    notFailBuild: true
        }
    }
}
