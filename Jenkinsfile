#!groovy

@Library('jenkins-pipeline-library') _

// Initialize global config
cfg = jplConfig('gp-mail', 'bash', '', [email: env.CI_NOTIFY_EMAIL_TARGETS])

/**
 * Build and publish docker images
 *
 * @param nextReleaseNumber String Release number to be used as tag
 */
def buildAndPublishDockerImage(nextReleaseNumber = "") {
    if (nextReleaseNumber == "") {
        nextReleaseNumber = jplGetNextReleaseNumber(cfg).substring(1)
    }
    docker.withRegistry("https://ghcr.io", 'docker-jenkins-ayudadigital') {
        def customImage = docker.build("${env.DOCKER_ORGANIZATION}/${cfg.projectName}:${nextReleaseNumber}", "--pull --no-cache ${cfg.projectName.substring(3)}")
        customImage.push()
        if (nextReleaseNumber != "beta") {
            customImage.push('latest')
        }
    }
}

pipeline {
    agent { label 'docker' }

    stages {
        stage ('Initialize') {
            steps  {
                jplStart(cfg)
            }
        }
        stage ('Bash linter') {
            steps {
                sh 'devcontrol run-bash-linter'
            }
        }
        stage ('Build') {
            steps {
                buildAndPublishDockerImage("beta")
            }
        }
        stage ('Make release') {
            when { branch 'release/new' }
            steps {
                buildAndPublishDockerImage()
                jplMakeRelease(cfg, true)
                deleteDir()
            }
        }
    }

    post {
        always {
            jplPostBuild(cfg)
        }
    }

    options {
        timestamps()
        ansiColor('xterm')
        buildDiscarder(logRotator(artifactNumToKeepStr: '20',artifactDaysToKeepStr: '30'))
        disableConcurrentBuilds()
        timeout(time: 10, unit: 'MINUTES')
    }
}
