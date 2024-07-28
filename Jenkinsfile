pipeline {
    agent any

    stages {
      // Here start the build stage
        stage('Build') {
          agent {
            docker {
              image 'node:18-alpine'
              reuseNode true
            }
          }
            steps {
                sh '''
                ls -la
                node --version
                npm --version
                npm ci
                npm run build
                ls -la
                '''
            }
        }

        stage('Tests') {
          parallel {
            stage('Unit Tests') {
              agent {
                docker {
                  image 'node:18-alpine'
                  reuseNode true
                }
              }

              steps {
                sh '''
                  echo Test stage
                  test -f build/index.html
                  npm run test
                '''
              }
                  post {
                    always {
                      junit 'jest-results/junit.xml'
                    }
                  }
            }
            stage('e2e') {
              agent {
                docker {
                  image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                  reuseNode true
                }
              }

              steps {
                sh '''
                  echo E2E Test
                  npm i serve
                  npx serve -s build &
                  sleep 10
                  npx playwright test --reporter=html
                '''
              }
              post {
                    always {
                      publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                    }
                  }
            }
          }
        }

        stage('Deploy') {
          agent {
            docker {
              image 'node:18-alpine'
              reuseNode true
            }
          }
            steps {
                sh '''
                  npm i netlify-cli
                  node_modules/.bin/netlify --version
                '''
            }
        }


    }
}