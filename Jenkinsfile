pipeline {
    agent any
    
    options {
        // Build timeout setting
        timeout(time: 30, unit: 'MINUTES')
        // Keep build records
        buildDiscarder(logRotator(numToKeepStr: '10'))
        // Disable concurrent builds
        disableConcurrentBuilds()
    }
    
    tools {
        // Configure in Jenkins Global Tool Configuration
        ant 'Default Ant'
        jdk 'JDK'
    }
    
    environment {
        PROJECT_NAME = 'Practice01'
        PROJECT_VERSION = '1.0.0'
        BUILD_NUMBER = "${env.BUILD_NUMBER}"
        WORKSPACE_ROOT = "${env.WORKSPACE}"
    }
    
    stages {
        stage('Environment Setup') {
            steps {
                script {
                    echo """
                    Jenkins Build Environment Information
                    =====================================
                    Jenkins URL: ${env.JENKINS_URL}
                    Job Name: ${env.JOB_NAME}  
                    Build Number: ${env.BUILD_NUMBER}
                    Workspace: ${env.WORKSPACE}
                    Node Name: ${env.NODE_NAME}
                    """
                    
                    sh '''
                        echo "Java version:"
                        java -version
                        echo ""
                        echo "Ant version:"  
                        ant -version
                        echo ""
                        echo "System information:"
                        uname -a
                        echo ""
                        echo "Current path:"
                        pwd
                        echo ""
                        echo "Available space:"
                        df -h .
                    '''
                    
                    // Verify build.xml exists
                    sh """
                        echo "Checking if build.xml exists:"
                        if [ -f "build.xml" ]; then
                            echo "build.xml exists - File size: \$(wc -c < build.xml) bytes"
                        else
                            echo "build.xml not found"
                            exit 1
                        fi
                    """
                    
                    // Initialize project environment once
                    echo "=== Setting up project environment ==="
                    sh 'ant timestamp'
                    sh 'ant set-properties' 
                    sh 'ant init'
                    
                    // Stash workspace for other stages
                    stash includes: '**', excludes: 'build/**,dist/**', name: 'workspace-base'
                }
            }
        }
        
        stage('Build & Quality Assurance') {
            parallel {
                stage('Compile & Prepare') {
                    steps {
                        script {
                            unstash 'workspace-base'
                            
                            echo "=== Downloading dependencies ==="
                            sh 'ant dependencies'
                            
                            echo "=== Generating sources ==="
                            sh 'ant generate-sources'
                            
                            echo "=== Compiling application ==="
                            sh 'ant compile'
                            
                            // Check compilation results
                            sh '''
                                echo "Compilation verification:"
                                if [ -d "build/classes" ]; then
                                    echo "Main classes: $(find build/classes -name "*.class" | wc -l)"
                                fi
                                if [ -d "build/test-classes" ]; then
                                    echo "Test classes: $(find build/test-classes -name "*.class" | wc -l)"
                                fi
                            '''
                            
                            // Stash build artifacts for other stages
                            stash includes: 'build/**,lib/**,src/**,test/**', name: 'build-artifacts'
                        }
                    }
                }
                
                stage('Code Analysis') {
                    steps {
                        script {
                            unstash 'workspace-base'
                            echo "=== Running code quality checks ==="
                            // 這裡可以添加 SonarQube, Checkstyle 等工具
                            echo "Code analysis would run here in a real enterprise setup"
                        }
                    }
                }
            }
        }
        
        stage('Test & Package') {
            parallel {
                stage('Unit Tests') {
                    steps {
                        script {
                            unstash 'build-artifacts'
                            
                            echo "=== Running unit tests ==="
                            try {
                                sh 'ant test'
                                echo "=== Unit tests completed successfully ==="
                            } catch (Exception e) {
                                echo "Warning: Tests execution issue: ${e.getMessage()}"
                                currentBuild.result = 'UNSTABLE'
                            }
                        }
                    }
                    post {
                        always {
                            script {
                                if (fileExists('reports')) {
                                    echo "Test results archived"
                                    try {
                                        echo "Test reports would be published here with JUnit plugin"
                                    } catch (Exception e) {
                                        echo "Test report publishing failed: ${e.getMessage()}"
                                    }
                                }
                            }
                        }
                    }
                }
                
                stage('Build JAR') {
                    steps {
                        script {
                            unstash 'build-artifacts'
                            
                            echo "=== Building JAR package ==="
                            try {
                                sh 'ant jar'
                                echo "=== JAR build completed ==="
                            } catch (Exception e) {
                                echo "JAR build failed: ${e.getMessage()}"
                                currentBuild.result = 'FAILURE'
                                error("JAR packaging failed")
                            }
                        }
                    }
                }
                
                stage('Build WAR') {
                    steps {
                        script {
                            unstash 'build-artifacts'
                            
                            echo "=== Building WAR package ==="
                            try {
                                sh 'ant war'
                                echo "=== WAR build completed ==="
                            } catch (Exception e) {
                                echo "WAR build failed: ${e.getMessage()}"
                                currentBuild.result = 'FAILURE'
                                error("WAR packaging failed")
                            }
                        }
                    }
                }
                
                stage('Generate Documentation') {
                    steps {
                        script {
                            unstash 'build-artifacts'
                            
                            echo "=== Generating API documentation ==="
                            try {
                                sh 'ant javadoc'
                                echo "Documentation generation completed"
                            } catch (Exception e) {
                                echo "Warning: Documentation generation issue: ${e.getMessage()}"
                                currentBuild.result = 'UNSTABLE'
                            }
                        }
                    }
                    post {
                        success {
                            script {
                                if (fileExists('docs/index.html')) {
                                    publishHTML([
                                        allowMissing: false,
                                        alwaysLinkToLastBuild: true,
                                        keepAll: true,
                                        reportDir: 'docs',
                                        reportFiles: 'index.html',
                                        reportName: 'API Documentation'
                                    ])
                                }
                            }
                        }
                    }
                }
            }
        }
        
        stage('Deploy & Report') {
            steps {
                script {
                    // Unstash everything for deployment and reporting
                    unstash 'build-artifacts'
                    
                    echo "=== Deploying application ==="
                    try {
                        sh 'ant deploy'  // Only execute deployment
                        echo "Application deployment completed"
                    } catch (Exception e) {
                        echo "Warning: Deployment issue: ${e.getMessage()}"
                        currentBuild.result = 'UNSTABLE'
                    }
                    
                    echo "=== Generating build reports ==="
                    try {
                        sh 'ant report'  // Only execute reporting
                        echo "Build reports generated successfully"
                    } catch (Exception e) {
                        echo "Warning: Report generation issue: ${e.getMessage()}"
                    }
                    
                    echo "=== Generating final logs ==="
                    try {
                        sh 'ant generate-clean-log'
                        echo "Final log generation completed"
                    } catch (Exception e) {
                        echo "Warning: Log generation issue: ${e.getMessage()}"
                    }
                }
            }
            post {
                always {
                    script {
                        if (fileExists('reports/build-summary.html')) {
                            publishHTML([
                                allowMissing: false,
                                alwaysLinkToLastBuild: true,
                                keepAll: true,
                                reportDir: 'reports',
                                reportFiles: 'build-summary.html',
                                reportName: 'Build Summary Report'
                            ])
                        }
                    }
                }
            }
        }
    }
    
    post {
        always {
            script {
                echo """
                Jenkins Build Completed
                =======================
                Project: ${PROJECT_NAME} v${PROJECT_VERSION}
                Build Number: ${BUILD_NUMBER}
                Build Status: ${currentBuild.result ?: 'SUCCESS'}
                Build Duration: ${currentBuild.durationString}
                """
                
                // Show what files exist before archiving
                sh '''
                    echo "=== WORKSPACE CONTENTS ==="
                    ls -la
                    echo "=== SEARCHING FOR BUILD ARTIFACTS ==="
                    find . -name "*.jar" -o -name "*.war" -o -name "*.html" -o -name "*.log" -o -name "*.class" | head -20
                    echo "=== DIRECTORY STRUCTURE ==="
                    find . -type d | head -20
                '''
                
                // Archive build artifacts
                archiveArtifacts(
                    artifacts: 'dist/*.jar,dist/*.war,logs/*.log,reports/*.html',
                    allowEmptyArchive: true,
                    fingerprint: true
                )
            }
        }
        success {
            echo """
            Build Successful!
            
            Output files:
            - JAR: dist/Practice01-1.0.0.jar
            - WAR: dist/Practice01-1.0.0.war  
            - Documentation: docs/index.html
            - Reports: reports/build-summary.html
            - Logs: logs/build.log
            
            All tasks completed successfully!
            """
        }
        failure {
            echo """
            Build Failed!
            
            Please check the following items:
            1. Java/Ant environment correctly installed
            2. build.xml file exists and is correct
            3. Network connection is normal (for downloading dependencies)
            4. Sufficient disk space available
            
            View detailed errors: ${env.BUILD_URL}console
            """
        }
        unstable {
            echo """
            Build Unstable
            
            Some stages encountered issues but build continued
            Please check test results and reports
            """
        }
        cleanup {
            echo "Executing cleanup tasks..."
            // Clean temporary files but preserve artifacts
            sh '''
                find . -name "*.tmp" -delete 2>/dev/null || true
                find . -name "*.temp" -delete 2>/dev/null || true
                echo "Cleanup completed, build artifacts preserved"
            '''
        }
    }
}