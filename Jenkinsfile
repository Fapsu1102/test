gitlink = 'https://git.viettelpost.vn/autotest/vtp_vtp.git'

def runTest(String testsuite, String environment) {
    try {
        stage("Test ${testsuite}") {
            // sh "echo Running on Node: ${env.NODE_NAME}"
            // sh "echo Executor Number: ${env.EXECUTOR_NUMBER}"
            // sh "echo Workspace: ${env.WORKSPACE}"
            sh "echo Kiểm thử tính năng ${testsuite}"
            sh "chmod +x '${env.WORKSPACE}/src/test/resources/webdriver/linux/chromedriver'"
            // sh "sleep 10"
            // sh "whoami"
            sh "/usr/bin/mvn clean verify '-Dserenity.outputDirectory=target/site/serenity/${testsuite}' -Denvironment=${environment} -Dtest=${testsuite} -Dwebdriver.chrome.whitelistedIps="        }
    }

    catch (e) {
        echo "Test ${testsuite} failed: ${e.toString()}"
    }

    finally {
        def current_date = new Date().format('yyyyMMdd')
        def report_directory = "/var/www/serenity-reports/${current_date}"

        stage("Publish ${testsuite} report") {
            sh "rsync -var ${env.WORKSPACE}/target/site/serenity/* jenkins@192.168.11.44:${report_directory}"
        }

        stage("Noti to telegram group") {
            def message = "Finish ${testsuite}. \n Report: https://dev-autotest.viettelpost.vn/${current_date}/${testsuite}"
            def payload = [
                chat_id: "-4680876558",
                text: message
            ]
            def jsonPayload = groovy.json.JsonOutput.toJson(payload)
            sh "curl -X POST -H 'Content-Type: application/json' -d '${jsonPayload}' https://api.telegram.org/bot7259604176:AAFcMswBMP5yUlG8zC-w1QdfijuXXVH-P2A/sendMessage"
        }
    }
}


def gitcheckout() {
    node('ktht-autotest1') { 
        stage('Checkout') {
            checkout([$class: 'GitSCM',
                      branches: [[name: '*/trungnxt']],
                      extensions: [[$class: 'CleanBeforeCheckout']],
                      userRemoteConfigs: [[credentialsId: 'anhtn56', url: gitlink]]])
            stash name: 'source-code', includes: '**/*'
        }
    }
    
}




def userInput = input(id: 'userInput',
    message: 'Enter comma-separated list of features:',
    parameters: [string(defaultValue: '', description: 'Comma-separated list of features', name: 'features'),])

def (environment, features) = userInput.tokenize(':').collect { it.trim() }
def featuresList = features.tokenize(',').collect { it.trim() }

println "Environment: ${environment}"
println "Features List: ${featuresList}"

gitcheckout()

def parallelStages = [:]

featuresList.each { feature ->
    parallelStages[feature] = {
        node('ktht-autotest1') {
            ws("${env.WORKSPACE}") {
                unstash 'source-code'
                runTest(feature.trim(), environment)
            }
        }
    }
}

parallel parallelStages