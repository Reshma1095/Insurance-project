node {
    def mavenHome
    def mavenCMD
    def docker
    def dockerCMD
    def tagName
    
    stage('Prepare Environment') {
        echo 'Initialize all the variables'
        mavenHome = tool name: 'maven', type: 'maven'
        mavenCMD = "${mavenHome}/bin/mvn"
        docker = tool name: 'docker', type: 'org.jenkinsci.plugins.docker.commons.tools.DockerTool'
        dockerCMD = "${docker}/bin/docker"
        tagName = "3.0"
    }
    
    stage('Git Code Checkout') {
        try {
            echo 'Checkout the code from git repository'
            git 'https://github.com/Reshma1095/Insurance-project.git'
        } catch (Exception e) {
            echo 'Exception occurred in Git Code Checkout Stage'
            currentBuild.result = "FAILURE"
            emailext body: '''Dear All,
            The Jenkins job ${JOB_NAME} has failed. Please have a look at it immediately by clicking on the link below.
            ${BUILD_URL}''', subject: 'Job ${JOB_NAME} ${BUILD_NUMBER} Failed', to: 'reshmajk1095@gmail.com'
            error "Stopping build due to Git checkout failure."
        }
    }
    
    stage('Build the Application') {
        echo "Cleaning, Compiling, Testing, and Packaging..."
        sh "${mavenCMD} clean package"        
    }
    
    stage('Publish Test Reports') {
        echo 'Publishing test reports'
        publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, 
                     reportDir: 'target/surefire-reports', reportFiles: 'index.html', 
                     reportName: 'HTML Report', useWrapperFileDirectly: true])
    }
    
    stage('Containerize the Application') {
        echo 'Creating Docker image'
        sh "${dockerCMD} build -t insuranceproject1/insure-me:${tagName} ."
    }
    
    stage('Push to DockerHub') {
        echo 'Pushing the Docker image to DockerHub'
        withCredentials([string(credentialsId: 'dock-password', variable: 'dockerHubPassword')]) {
            sh "${dockerCMD} login -u insuranceproject1 -p ${dockerHubPassword}"
            sh "${dockerCMD} push insuranceproject1/insure-me:${tagName}"
        }
    }

    stage('Configure and Deploy to Test-Server') {
        echo 'Deploying application to test server'
        ansiblePlaybook become: true, credentialsId: 'ansible-key', disableHostKeyChecking: true, 
                       installation: 'ansible', inventory: '/etc/ansible/hosts', playbook: 'ansible-playbook.yml'
    }
}
