pipeline{
agent any

parameters{
    string(name: 'SPEC', defaultValue:"cypress/e2e/1-getting-started/todo.cy.js", description: "Enter the cypress script path that you want to execute")
    choice(name: 'BROWSER', choices:['electron', 'chrome', 'edge', 'firefox'], description: "Select the browser to be used in your cypress tests")
}

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
