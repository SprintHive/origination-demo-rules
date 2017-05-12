#!/usr/bin/groovy
@Library('github.com/fabric8io/fabric8-pipeline-library@v2.2.311')
@Library('github.com/SprintHive/sprinthive-pipeline-library@debug')

def componentName = 'origination-demo-rules'
def newVersion = ''

mavenNode(label: 'maven-and-docker') {
    stage('Compile source') {
        checkout scm

        container(name: 'maven') {
            sh 'mvn clean package'
        }
    }

    stage('Build and publish docker image') {
        newVersion = publishDockerImage{
            name = componentName
        }
    }

    stage('Create deployment objects') {
        def rc = buildKubernetesObjects{
          port = 8080
          icon = '/img/icon.png'
          version = newVersion
          name = componentName
          stage = 'testing'
        }
    }

    stage('Rollout to Testing') {
        def envStage = 'origination-demo-testing'

        // Launch rules
        kubernetesApply(file: readFile('src/main/resources/data/drools-workbench-deployment.json'), environment: envStage)
        kubernetesApply(file: readFile('src/main/resources/data/drools-workbench-service.json'), environment: envStage)
        kubernetesApply(file: readFile('src/main/resources/data/kie-server-deployment.json'), environment: envStage)
        kubernetesApply(file: readFile('src/main/resources/data/kie-server-service.json'), environment: envStage)

        // Launch Service
        kubernetesApply(file: rc, environment: envStage)
    }
}

