pipeline {
    agent none

    environment {
      VERSION = "1.0"
      APP_NAME = "devops-demo"
      BUILD_DIR = "build"
    }

    parameters {
      string (
        name: 'GIT_BRANCH',
        defaultValue: 'main',
        description: 'Git branch to checkout'
      )

      choice (
        name: 'ENVIRONMENT',
        choices: ['dev', 'test', 'prod'],
        description: 'Select deployment environment'
      )

      booleanParam (
        name: 'RUN_TESTS',
        defaultValue: true,
        description: 'Run test atage?'
      )
    }

    stages {
        stage("Checkout") {
          agent { label 'jenkins-git' }
          steps {
            echo "Checkout branch: ${params.GIT_BRANCH}"
            git branch: "${params.GIT_BRANCH}",
                url: 'https://github.com/GayanaJinde/devops-learning' // public repo, so no credentialsId required
            
            stash name: 'source-code',
                  includes: '**',
                  excludes: '.git/**'
          }
        }

        stage("Build") {
          agent { label 'jenkins-git' }
          steps {

            unstash 'source-code'

            echo "Building ${env.APP_NAME}"

            sh '''
              echo "Starting build..."
              mkdir -p ${BUILD_DIR}
              find . -type f -not -path "./build/*" | wc -l > ${BUILD_DIR}/file_count.txt

              echo "Application Version: ${VERSION}" > ${BUILD_DIR}/app.txt
              echo "Build Successful" >> ${BUILD_DIR}/app.txt
            '''

            //save build output for another agent
            stash name: 'build-output',
                  includes: '**',
                  excludes: '.git/**'
          }
        }

        stage("Test") {

          agent { label 'ubuntu' }
          
          //runs when it is true
          when {
            expression {
              return params.RUN_TESTS
            }
          }
          steps {
            echo "Running Tests..."
            
            //Bring Source's and Build's output into this agent
            unstash 'source-for-test'
            unstash 'build-files'
            
            sh '''
              BUILD_COUNT=$(cat build/file_count.txt)
              CURRENT_COUNT=$(find . -type f -not -path "./build/*" | wc -l)

              echo "Build counted: $BUILD_COUNT files"
              echo "Test counted: $CURRENT_COUNT files"

              if [ "$BUILD_COUNT" -eq "$CURRENT_COUNT" ]; then
                echo "TEST PASSED"
                exit 0
              else
                echo "TEST FAILED"
                exit 1
              fi
          '''
        }
      }

      stage('Deploy') {
        agent { label 'ubuntu'}

        steps {
          sh ''' 
            echo "Deploying application..."
            echo "Selected environment: ${ENVIRONMENT}"

            if [ "${ENVIRONMENT}" = "dev" ]; then
              echo "Deploying to DEV"
            elif [ "${ENVIRONMENT}" = "test" ]; then
              echo "Deploying to TEST"
            else
              echo "Deploying to PRODUCTION"
            fi
          '''
        }
      }
  }

  post {
    success {
      echo "Pipeline completed successfully"
      mail (
        to: 'gayanajinde@gmail.com',
        subject: "SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
        body: "Pipeline completed successfully. Build URL: ${env.BUILD_URL}"
      )
    }
    failure {
      echo "Pipeline failed"
      mail (
        to: 'gayanajinde@gmail.com',
        subject: "FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
        body: "Pipeline Failed. Check Jenkins: ${env.BUILD_URL}"
      )
    }
  }
}
