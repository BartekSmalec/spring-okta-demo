node {
    def repourl = "${REGISTRY_URL}/${PROJECT_ID}/${ARTIFACT_REGISTRY}"
    def mvnHome = tool name: 'maven', type: 'maven'
    def mvnCMD = "${mvnHome}/bin/mvn "
    stage('Checkout') {
        checkout([$class: 'GitSCM',
            branches: [[name: '*/main']],
            extensions: [],
            userRemoteConfigs: [[credentialsId: 'git',
            url: 'https://github.com/BartekSmalec/spring-okta-demo.git']]])
    }
    stage('Build and Push Image') {
        withCredentials([file(credentialsId: 'gcp', variable: 'GC_KEY')]) {
            sh("gcloud auth activate-service-account --key-file=${GC_KEY} --account=jenkins@cool-eye-394414.iam.gserviceaccount.com")
            sh 'gcloud auth configure-docker  us-central1-docker.pkg.dev'
            sh "${mvnCMD} clean install jib:build -DREPO_URL=${REGISTRY_URL}/${PROJECT_ID}/${ARTIFACT_REGISTRY}"
        }
    }
    stage('Deploy') {
        sh "sed -i 's|IMAGE_URL|${repourl}|g' k8s/deployment.yaml"
        step([$class: 'KubernetesEngineBuilder',
            projectId: env.PROJECT_ID,
            clusterName: env.CLUSTER,
            location: env.ZONE,
            manifestPattern: 'k8s/deployment.yaml',
            credentialsId: env.PROJECT_ID,
            verifyDeployments: true])
    }
}