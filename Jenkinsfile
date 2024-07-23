pipeline {
    agent none
    stages {
        stage('Checkout') {
            agent any
            steps {
                git branch: 'master', url: 'https://github.com/OWASP/Vulnerable-Web-Application.git'
            }
        }
        stage('OWASP DependencyCheck') {
            agent any
            steps {
                dependencyCheck additionalArguments: '--format HTML --format XML', odcInstallation: 'OWASP-Dependency-Check'
            }
            post {
                success {
                    dependencyCheckPublisher pattern: 'dependency-check-report.xml'
                }
            }
        }
        stage('Build Project') {
            agent any
            steps {
                sh '/var/jenkins_home/apache-maven-3.6.3/bin/mvn --batch-mode -V -U -e clean verify -Dsurefire.useFile=false -Dmaven.test.failure.ignore'
            }
        }
        stage('Analysis') {
            agent any
            steps {
                sh '/var/jenkins_home/apache-maven-3.6.3/bin/mvn --batch-mode -V -U -e checkstyle:checkstyle pmd:pmd pmd:cpd findbugs:findbugs'
            }
            post {
                always {
                    junit testResults: '**/target/surefire-reports/TEST-*.xml'
                    recordIssues enabledForFailure: true, tools: [mavenConsole(), java(), javaDoc()]
                    recordIssues enabledForFailure: true, tool: checkStyle()
                    recordIssues enabledForFailure: true, tool: spotBugs(pattern: '**/target/findbugsXml.xml')
                    recordIssues enabledForFailure: true, tool: cpd(pattern: '**/target/cpd.xml')
                    recordIssues enabledForFailure: true, tool: pmdParser(pattern: '**/target/pmd.xml')
                }
            }
        }
        stage('Code Quality Check via SonarQube') {
            agent any
            steps {
                script {
                    def scannerHome = tool 'SonarQube'
                    withSonarQubeEnv('SonarQube') {
                        sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=YourProjectKey -Dsonar.sources=."
                    }
                }
            }
            post {
                always {
                    recordIssues enabledForFailure: true, tool: sonarQube()
                }
            }
        }
    }
}
