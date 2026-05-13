@Library('jenkins-helpers') _

properties([
    disableConcurrentBuilds(abortPrevious: env.BRANCH_NAME != 'master'),
    disableResume(),
])

def podTemplates = { body ->
    podTemplate(
        annotations: [
            podAnnotation(key: "jenkins/build-url", value: env.BUILD_URL ?: ""),
            podAnnotation(key: "jenkins/github-pr-url", value: env.CHANGE_URL ?: ""),
        ],
        containers: [
            containerTemplate(
                name: 'protoc',
                image: 'alpine:3.23.4',
                command: 'cat',
                ttyEnabled: true,
                resourceRequestCpu: '100m',
                resourceRequestMemory: '128Mi',
            ),
        ]) {
        node(POD_LABEL) {
            timeout(time: 15, unit: 'MINUTES') {
                body()
            }
        }
    }
}

podTemplates {
    def isMaster = env.BRANCH_NAME == 'master'
    try {
        stage('Checkout') {
            checkout(scm)
        }
        stage('Verify proto syntax') {
            container('protoc') {
                sh 'apk add --no-cache protobuf'
                sh 'mkdir -p kotlin_out'
                dir("v1/timeseries") {
                    sh 'find . -name "*.proto" | xargs protoc --proto_path=. --kotlin_out=kotlin_out'
                }
            }
        }
    } catch (e) {
        currentBuild.result = 'FAILURE'
        throw e
    } finally {
        if (isMaster && currentBuild.result == 'FAILURE') {
            slackSend(channel: "#alerts-timelords-jenkins", color: "danger", message: "[${currentBuild.result}] *protobuf-files* ${env.BRANCH_NAME} build (<${env.RUN_DISPLAY_URL}|Open>)")
        }
    }
}
