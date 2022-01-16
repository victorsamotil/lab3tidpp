pipeline 
{
    agent any

    tools 
    {
        maven "M3"
        jdk "JDK10"
    }
    
    parameters
    {
        booleanParam(name: 'CLEAN_WORKSPACE', defaultValue: true, description: 'To delete build folder')
    }
    
    environment
    {
        GOOGLE_API_KEY = "AIzaSyA-DCgVotefXQ9QVG3MvMsnJvKBzNkHi50"
        DOCKER_CREDENTIAL = credentials('dockerhub-login')
        ON_FAILURE_SEND_EMAIL = true
        ON_SUCCESS_SEND_EMAIL = true
        TESTING_FRONTEND = false
    }
    
    stages 
    {
        stage('Build') 
        {
            steps 
            {
                // Get some code from a GitHub repository
                git branch: 'main', poll: false, url: 'https://github.com/victorsamotil/lab3tidpp.git'

                // Run Maven on a Unix agent.
                // "mvn -Dmaven.test.failure.ignore=true clean package"

                // To run Maven on a Windows agent, use
                // bat "mvn -Dmaven.test.failure.ignore=true clean package"
                bat "mvn clean package"
                // bat "mvn spring-boot:run"
            }

            post 
            {
                // If Maven was able to run the tests, even if some of the test
                // failed, record the test results and archive the jar file.
                success
                {
                    echo "Updates installed successful."
                    //junit '**/target/surefire-reports/TEST-*.xml'
                    junit skipPublishingChecks: true, testResults: '**/target/surefire-reports/TEST-*.xml'
                    // archiveArtifacts 'target/*.jar'
                    // cleanWs()
                }
                failure
                {
                    echo "Critical failure, you noob !!!"
                }
            }
        }
        
        stage('Backend Test')
        {
            steps
            {
                echo "Backend test stage: execution."
            }
            post
            {
                success
                {
                    script
                    {
                        echo "Backend test stage: successful."
                        TESTING_FRONTEND = true
                        ON_FAILURE_SEND_EMAIL = false
                    }
                }
                failure
                {
                    script
                    {
                        echo "Backend test stage: failed."
                        TESTING_FRONTEND = false
                        ON_SUCCESS_SEND_EMAIL = false
                    }
                }
            }
        }
        
        stage('Frontend Test')
        {
            steps
            {
                echo "Frontend test stage: ${TESTING_FRONTEND}"
            }
        }
        
        stage('Continuous Delivery')
        {
            steps
            {
                echo "${DOCKER_CREDENTIAL_USR}"
                echo "${DOCKER_CREDENTIAL_PSW}"
                bat "docker build -t video-service ."
                bat "docker tag video-service:latest victorsamotil1/lab4"
                bat "docker login -u ${DOCKER_CREDENTIAL_USR} -p ${DOCKER_CREDENTIAL_PSW} docker.io"
                bat "docker push victorsamotil1/lab4"
                bat "docker rmi victorsamotil1/lab4"
            }
        }
        
        stage('Continuous Deployment')
        {
            steps
            {
                echo "SSH into VBox..."
                bat "ssh -p 1337 victor@127.0.0.1 \"cd /home/victor/tidpplab && docker-compose up -d\""
            }
        }
    }

    post
    {
        always
        {
            echo "Pipeline execution has ended."
        }
            
        success
        {
            script
            {
                echo "Success !!!"
                if (env.ON_SUCCESS_SEND_EMAIL)
                {
                     emailext(body: "Job name: ${JOB_NAME}, build number: ${BUILD_NUMBER} - Successful! Build URL: ${BUILD_URL}", subject: 'Build success', to: 'victor.samotil1@gmail.com')
                }
                if (params.CLEAN_WORKSPACE == true)
                {
                    bat "if exist ${BUILD_TAG} rmdir ${BUILD_TAG} /q /s"
                    echo "Folder /builds/${BUILD_TAG} has been deleted."
                }
                else
                {
                    echo "Folder /builds/${BUILD_TAG} doesn't exist."
                }
                cleanWs()
                echo "Workspace has been cleaned."
            }
        }
            
        failure
        {
            script
            {
                echo "Your code sucks !!!"
                if (env.ON_FAILURE_SEND_EMAIL)
                {
                     emailext(body: "Job name: ${JOB_NAME}, build number: ${BUILD_NUMBER} - Failure! Build URL: ${BUILD_URL}", subject: 'Build fail', to: 'victor.samotil1@gmail.com')
                }
                cleanWs()
                echo "Workspace has been cleaned."
            }
        }
        
        unstable
        {
            echo "Idk man, there's problems, but I kinda can't put my finger on it."
        }
    }
}
