pipeline 
{
    agent any

    environment
    {
        NETLIFY_SITE_ID = '0d0293c4-93b6-4a7b-8419-625129641d72'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

    stages 
    {
        stage('Build') 
        {
            agent
            {
                docker
                {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps 
            {
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
        
        stage('Run Tests')
        {
            parallel
            {
            stage('Test')
        {
            agent
            {
                docker
                {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps
            {
                echo 'Testing stage'
                sh 'test -f build/index.html'
                sh 'npm test'
            }
        }
        stage('E2E')
        {
            agent
            {
                docker
                {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true 
                }
            }
            steps
            {
                sh 'npm install serve'
                sh 'node_modules/.bin/serve -s build &'
                sleep(10)
                sh 'npx playwright test --reporter=html'
            }
             post
            {
                always
                {
                    junit 'jest-results/junit.xml'
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Local', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
            }
        }
        stage('Deploy') 
        {
            agent
            {
                docker
                {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps 
            {
                sh '''
                    npm install netlify-cli
                    node_modules/.bin/netlify --version
                    echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --prod
                '''
            }
        }
        stage('Prod E2E')
        {
            agent
            {
                docker
                {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true 
                }
            }

            environment
            {
                CI_ENVIRONMENT_URL = 'https://gorgeous-tarsier-5a08e2.netlify.app'
            }
            steps
            {
                sh 'npx playwright test --reporter=html'
            }
        }
    }
    post
    {
        always
        {
            junit 'jest-results/junit.xml'
            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright E2E', reportTitles: '', useWrapperFileDirectly: true])
        }
    }
    
}