pipeline {
     // 전역 에이전트를 사용하지 않음으로써 컨테이너 중첩 방지
    agent none 

    environment {
        NETLIFY_SITE_ID = 'eddd4f00-388e-4cd2-b0a7-dafdd66e029e'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

    stages {
        stage('Build') {
            agent {
                docker { 
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo '빌드 시작..'
                    node --version
                    npm --version
                    npm ci
                    npm run build
                '''
            }
        }

        stage('AWS') {
            agent {
                docker { 
                    image 'amazon/aws-cli'
                    reuseNode true
                    args "--entrypoint=''" 
                }
            }

            environment {
                AWS_S3_BUCKET = 'learn-jenkins-202606301126'
            }

            steps {
                withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                        aws --version
                        aws s3 sync build s3://$AWS_S3_BUCKET
                    '''
                }
            }
        }

        stage('Test') {
            agent {
                docker { image 'mcr.microsoft.com/playwright:v1.39.0-jammy' }
            }
            steps {
                sh '''
                    npm test
                '''
            }
        }

        stage('E2E') {
            agent {
                docker { image 'mcr.microsoft.com/playwright:v1.39.0-jammy' }
            }
            steps {
                sh '''
                    # serve를 로컬에 설치하여 실행
                    npm install serve
                    node_modules/.bin/serve -s build & sleep 10
                    npx playwright test --reporter=html
                '''
            }
        }

        stage('Deploy staging') {
            agent {
                docker { image 'node:18-bullseye' } 
            }
            steps {
                sh '''
                    npm install netlify-cli@20.1.1
                    node_modules/.bin/netlify deploy --dir=build
                '''
            }
        }

        stage('Approval'){
            agent none
            steps {
                timeout(time: 15, unit: 'MINUTES') {
                    input message: '운영환경에 배포할까요?', ok: '네 배포합니다'
                }
            }
        }

        stage('Deploy prod') {
            agent {
                docker { image 'node:18-bullseye' }
            }
            steps {
                sh '''
                    npm install netlify-cli@20.1.1
                    node_modules/.bin/netlify deploy --dir=build --prod
                '''
            }
        }

        stage('Prod E2E') {
            agent {
                docker { image 'mcr.microsoft.com/playwright:v1.39.0-jammy' }
            }
            environment {
                CI_ENVIRONMENT_URL = 'https://deluxe-tanuki-39fc61.netlify.app'
            }
            steps {
                sh 'npx playwright test --reporter=html'
            }
        }
    }

   
}
