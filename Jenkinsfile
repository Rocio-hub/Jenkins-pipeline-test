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
                sh "docker network create SE3"
                sh "docker run -d --rm -p 4444:4444 --net=SE3 --name selenium-hub3 selenium/hub"
                sh "docker run -d --rm --net=SE3 -e HUB_HOST=selenium-hub3 --name selenium-node-chrome selenium/node-chrome"
                sh "docker run -d --rm --net=SE3 -e HUB_HOST=selenium-hub3 --name selenium-node-firefox selenium/node-firefox"
                sh "docker run -d --rm --net=SE3 --name assignment-container roci0055/frontend-calc"                
            }
        }
        stage("Execute system tests") {
            steps {
                sh "selenium-side-runner --server http://localhost:4444/wd/hub -c 'browserName=firefox' test/system/FunctionalTests.side --base-url http://assignment-container" 
                sh "selenium-side-runner --server http://localhost:4444/wd/hub -c 'browserName=chrome' test/system/FunctionalTests.side --base-url http://assignment-container"
            }
        }
    }
    post {
        cleanup {
            echo "Cleaning the Docker environment"
            sh script:"docker stop selenium-hub3", returnStatus:true
            sh script:"docker stop selenium-node-firefox", returnStatus:true
            sh script:"docker stop selenium-node-chrome", returnStatus:true
            sh script:"docker stop assignment-container", returnStatus:true
            sh script:"docker network remove SE3", returnStatus:true
        }
    }
}    