pipeline {
    agent { label '' }
    environment {
        registry = "docker.io/dickydechmp/bpfe"
        serviceName= 'frontend_bp_trunkbase'
        dockerImage = ''
        branchname = "master"
    }
    stages {
        stage("Clone Code") {
            steps {
                script {
                checkout scm: [$class: 'GitSCM', userRemoteConfigs: [[url: 'https://github.com/dickydechamp/Bpfe.git',
                credentialsId: '1']], branches: [[name: "${branchname}" ]]],poll: true
                env.HASH = sh(script: "echo \$(git rev-parse --short HEAD)",returnStdout: true).trim()
                env.VERSION = "${env.BUILD_NUMBER}-${branchname}-${env.HASH}"
                }
            }
        }
        stage('Building Image') {
            steps{
                script {
                    sh "docker build -t $registry:$env.VERSION ."
                }
            }
        }
        stage('Push Image') {
            steps{ 
                script {
                    sh "docker push $registry:$env.VERSION"
                    }
                }
            }
        stage('Remove Unused Docker Image') {
            steps{
                script {
                    sh "docker rmi $registry:$env.VERSION"
                }
            }
        }
        stage('Deploy to Bpfe kubernetes staging') {
            steps{
                script {
                        sh "sed -i 's/bpfe:latest/bpfe:$env.VERSION/g' kubernetes/staging/deployment.yaml"
                        sh "kubectl apply -f kubernetes/staging/deployment.yaml"
                        sh "kubectl rollout status deployment -n production frontend-bp"
                        sh "kubectl get pods -n production | grep frontend-bp"
                }
            }
        }
        stage('Approval') {
            // no agent, so executors are not used up when waiting for approvals
            agent none
            steps {
                script {
                    def deploymentDelay = input id: 'Deploy', message: 'Deploy to production?', submitter: 'admin', parameters: [choice(choices: ['0', '1', '2', '3', '4', '5', '6', '7', '8', '9', '10', '11', '12', '13', '14', '15', '16', '17', '18', '19', '20', '21', '22', '23', '24'], description: 'Hours to delay deployment?', name: 'deploymentDelay')]
                    sleep time: deploymentDelay.toInteger(), unit: 'HOURS'
                }
            }
        }
        stage('Deploy to Production') {
            steps{
                script {
                        sh "sed -i 's/bpfe:latest/bpbe:$env.VERSION/g' kubernetes/production/deployment.yaml"
                        sh "kubectl apply -f kubernetes/production/deployment.yaml"
                        sh "kubectl rollout status deployment frontend-bp -n production"
                        sh "kubectl get pods -n production | grep frontend-bp"
                     }
            }
        }
    }
}
