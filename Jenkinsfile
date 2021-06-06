pipeline{
    agent any
    triggers {
        cron("*/5 * * * *")
    }
    stages {
        stage("Deliver to Docker Hub") {
            steps {
                sh "docker build . -t roci0055/frontend-calc"
                withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'DockerhubRo', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']])
                {
                    sh 'docker login -u ${USERNAME} -p ${PASSWORD}'
                }
                sh "docker push roci0055/frontend-calc"
            }
        }
        stage("Selenium grid setup") {
            steps {
                sh "docker network create SE"
                sh "docker run -d  -p 4444:4444 --net=SE --name selenium-hub selenium/hub"
                sh "docker run -d --rm --net=SE --name assignment-container roci0055/frontend-calc"
                sh "docker run -d --rm --net=SE -e HUB_HOST=selenium-hub --name selenium-node-chrome"
                sh "docker run -d --rm --net=SE -e HUB_HOST=selenium-hub --name selenium-node-firefox"                
            }
        }
        stage("Execute system tests") {
            steps {
                sh "selenium-side-runner --server http://localhost:4444/wd/hub -c 'browserName=firefox' --base-url http://assignment-container test/system/FunctionalTests.side"
                sh "selenium-side-runner --server http://localhost:4444/wd/hub -c 'browserName=chrome' --base-url http://assignment-container test/system/FunctionalTests.side"
            }
        }
    }
    post {
        cleanup {
            echo "Cleaning the Docker environment"
            sh script:"docker stop selenium-hub", returnStatus:true
            sh script:"docker stop selenium-node-firefox", returnStatus:true
            sh script:"docker stop selenium-node-chrome", returnStatus:true
            sh script:"docker stop assignment-container", returnStatus:true
            sh script:"docker network remove SE", returnStatus:true
        }
    }
}    