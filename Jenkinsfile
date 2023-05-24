// Example Jenkins pipeline with Cypress end-to-end tests running in parallel on 2 workers
// Pipeline syntax from https://jenkins.io/doc/book/pipeline/

// Setup:
//  before starting Jenkins, I have created several volumes to cache
//  Jenkins configuration, NPM modules and Cypress binary

// docker volume create jenkins-data
// docker volume create npm-cache
// docker volume create cypress-cache

// Start Jenkins command line by line:
//  - run as "root" user (insecure, contact your admin to configure user and groups!)
//  - run Docker in disconnected mode
//  - name running container "blue-ocean"
//  - map port 8080 with Jenkins UI
//  - map volumes for Jenkins data, NPM and Cypress caches
//  - pass Docker socket which allows Jenkins to start worker containers
//  - download and execute the latest BlueOcean Docker image

// docker run \
//   -u root \
//   -d \
//   --name blue-ocean \
//   -p 8080:8080 \
//   -v jenkins-data:/var/jenkins_home \
//   -v npm-cache:/root/.npm \
//   -v cypress-cache:/root/.cache \
//   -v /var/run/docker.sock:/var/run/docker.sock \
//   jenkinsci/blueocean:latest

// If you start for the very first time, inspect the logs from the running container
// to see Administrator password - you will need it to configure Jenkins via localhost:8080 UI
//    docker logs blue-ocean

pipeline{
agent any
stages {
stage('Build/Deploy app to staging') {
steps {
    sshPublisher(
        publishers: [
            sshPublisherDesc(
                configName: 'Staging', 
                transfers: [
                    sshTransfer(
                        cleanRemote: false, 
                        excludes: 'node_modules/,cypress/,**/*.yml',
                        execCommand: '''
                        cd /usr/share/nginx/html
                        npm ci
                        pm2 restart todo''',
                        execTimeout: 120000, 
                        flatten: false, 
                        makeEmptyDirs: false, 
                        noDefaultExcludes: false, 
                        patternSeparator: '[, ]+', 
                        remoteDirectory: '', 
                        remoteDirectorySDF: false, 
                        removePrefix: '', 
                        sourceFiles: '**/*'
                        )
                    ], 
                usePromotionTimestamp: false, 
                useWorkspaceInPromotion: false, 
                verbose: true
            )
        ]
    )
}
}
stage('Run automated tests'){
    when { expression { params.skip_test != true } }
    steps {
        echo "Running automated tests"
        sh 'npm prune'
        sh 'npm cache clean --force'
        sh 'npm i'
        sh 'npm install --save-dev mochawesome mochawesome-merge mochawesome-report-generator'
        sh 'npm run e2e:staging1spec'
    }
    post {
        success {
            publishHTML (
                target : [
                    allowMissing: false,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: 'mochawesome-report',
                    reportFiles: 'mochawesome.html',
                    reportName: 'My Reports',
                    reportTitles: 'The Report'])

        }
    }
}
stage('Perform manual testing') {
steps {
echo 'Performing manual testing'
}
}
stage('Release to production') {
steps {
echo 'Releasing to production'
}
}
}
}
