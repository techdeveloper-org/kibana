pipeline {
    agent {
        kubernetes {
            cloud 'kubernetes'
            inheritFrom 'k8s-agent'
            defaultContainer 'maven'
        }
    }



    environment {
        SERVICE_NAME = 'kibana'
        NAMESPACE = 'common'
        KUBECONFIG_UNIX = '/var/jenkins_home/.kube/config'
        KUBECONFIG_WIN = 'C:/Users/techd/.kube/config'
        DEPLOYMENT_TIMEOUT = '300s'
        SLACK_CHANNEL = '#deployments'
    }

    stages {
        stage('Pre-Deployment Validation') {
            steps {
                script {
                    echo "🔍 Validating K8s manifests for ${SERVICE_NAME}..."
                    def kubeconfig = isUnix() ? env.KUBECONFIG_UNIX : env.KUBECONFIG_WIN

                    withEnv(["KUBECONFIG=${kubeconfig}"]) {
                        if (isUnix()) {
                            sh """
                                echo "Checking kubectl connectivity..."
                                kubectl cluster-info || exit 1

                                echo "Validating K8s manifests..."
                                kubectl apply -f k8s/ --dry-run=client || exit 1
                            """
                        } else {
                            bat """
                                echo Checking kubectl connectivity...
                                kubectl cluster-info || exit /b 1

                                echo Validating K8s manifests...
                                kubectl apply -f k8s/ --dry-run=client || exit /b 1
                            """
                        }
                    }
                    echo "✅ Validation passed"
                }
            }
        }

        stage('Deploy to K8s') {

            steps {
                script {
                    echo "🚀 Deploying ${SERVICE_NAME} to namespace: ${NAMESPACE}"
                    def kubeconfig = isUnix() ? env.KUBECONFIG_UNIX : env.KUBECONFIG_WIN

                    withEnv(["KUBECONFIG=${kubeconfig}"]) {
                        try {
                            if (isUnix()) {
                                sh """


                                    # Apply K8s manifests
                                    kubectl apply -f k8s/

                                    echo "Deployment initiated successfully"
                                """
                            } else {
                                bat """


                                    REM Apply K8s manifests
                                    kubectl apply -f k8s/

                                    echo Deployment initiated successfully
                                """
                            }
                            echo "✅ Manifests applied successfully"
                        } catch (Exception e) {
                            echo "❌ Deployment failed: ${e.message}"
                            currentBuild.result = 'FAILURE'
                            error("Deployment failed")
                        }
                    }
                }
            }
        }

        stage('Verify Rollout Status') {
            when {
                expression { return !params.SKIP_DEPLOY }
            }
            steps {
                script {
                    echo "Deployment triggered. Pod will start on its own."
                    echo "Check rollout: kubectl rollout status deployment/${SERVICE_NAME} -n ${NAMESPACE}"
                    echo "Check logs: kubectl logs -l app=${SERVICE_NAME} -n ${NAMESPACE} --tail=100 -f"
                }
            }
        }

        stage('Health Check') {

            steps {
                script {
                    echo "🏥 Performing health checks..."
                    def kubeconfig = isUnix() ? env.KUBECONFIG_UNIX : env.KUBECONFIG_WIN

                    withEnv(["KUBECONFIG=${kubeconfig}"]) {
                        try {
                            if (isUnix()) {
                                sh """
                                    # Check pod status
                                    echo "Checking pod status..."
                                    kubectl get pods -n ${NAMESPACE} -l app=${SERVICE_NAME} || exit 1

                                    # Verify pods are ready
                                    READY_PODS=\$(kubectl get pods -n ${NAMESPACE} -l app=${SERVICE_NAME} -o jsonpath='{.items[*].status.conditions[?(@.type=="Ready")].status}' | grep -o "True" | wc -l)
                                    TOTAL_PODS=\$(kubectl get pods -n ${NAMESPACE} -l app=${SERVICE_NAME} --no-headers | wc -l)

                                    echo "Ready pods: \$READY_PODS / \$TOTAL_PODS"

                                    if [ "\$READY_PODS" -eq "\$TOTAL_PODS" ] && [ "\$TOTAL_PODS" -gt 0 ]; then
                                        echo "All pods are ready"
                                    else
                                        echo "Not all pods are ready"
                                        exit 1
                                    fi

                                    # Check service
                                    echo "Checking service..."
                                    kubectl get svc ${SERVICE_NAME} -n ${NAMESPACE} || exit 1
                                """
                            } else {
                                bat """
                                    REM Check pod status
                                    echo Checking pod status...
                                    kubectl get pods -n ${NAMESPACE} -l app=${SERVICE_NAME} || exit /b 1

                                    REM Check service
                                    echo Checking service...
                                    kubectl get svc ${SERVICE_NAME} -n ${NAMESPACE} || exit /b 1

                                    echo Health check completed
                                """
                            }
                            echo "✅ Health checks passed"
                        } catch (Exception e) {
                            echo "❌ Health check failed: ${e.message}"
                            currentBuild.result = 'UNSTABLE'
                            echo "⚠️ Deployment completed but health checks failed"
                        }
                    }
                }
            }
        }
        */

        stage('Deployment Summary') {

            steps {
                script {
                    echo "📊 Deployment Summary for ${SERVICE_NAME}"
                    def kubeconfig = isUnix() ? env.KUBECONFIG_UNIX : env.KUBECONFIG_WIN

                    withEnv(["KUBECONFIG=${kubeconfig}"]) {
                        if (isUnix()) {
                            sh """
                                echo "================================="
                                echo "Service: ${SERVICE_NAME}"
                                echo "Namespace: ${NAMESPACE}"
                                echo "================================="

                                echo "\\nDeployment Details:"
                                kubectl describe deployment ${SERVICE_NAME} -n ${NAMESPACE} | head -20

                                echo "\\nPod Status:"
                                kubectl get pods -n ${NAMESPACE} -l app=${SERVICE_NAME}

                                echo "\\nService Details:"
                                kubectl get svc ${SERVICE_NAME} -n ${NAMESPACE}
                            """
                        } else {
                            bat """
                                echo =================================
                                echo Service: ${SERVICE_NAME}
                                echo Namespace: ${NAMESPACE}
                                echo =================================

                                echo.
                                echo Pod Status:
                                kubectl get pods -n ${NAMESPACE} -l app=${SERVICE_NAME}

                                echo.
                                echo Service Details:
                                kubectl get svc ${SERVICE_NAME} -n ${NAMESPACE}
                            """
                        }
                    }
                }
            }
        }
    }

    post {
        success {
            script {
                echo "✅ Deployment SUCCEEDED for ${SERVICE_NAME}"
                echo "Build: ${env.BUILD_NUMBER}"
                echo "Status: SUCCESS"
            }
        }
        failure {
            script {
                echo "❌ Deployment FAILED for ${SERVICE_NAME}"
                echo "Build: ${env.BUILD_NUMBER}"
                echo "Status: FAILURE"

                def kubeconfig = isUnix() ? env.KUBECONFIG_UNIX : env.KUBECONFIG_WIN
                withEnv(["KUBECONFIG=${kubeconfig}"]) {
                    try {
                        echo "🔄 Attempting automatic rollback..."
                        if (isUnix()) {
                            sh """
                                kubectl rollout undo deployment/${SERVICE_NAME} -n ${NAMESPACE} || true
                                echo "Rollback initiated"
                            """
                        } else {
                            bat """
                                kubectl rollout undo deployment/${SERVICE_NAME} -n ${NAMESPACE}
                                echo Rollback initiated
                            """
                        }
                        echo "⚠️ Rolled back to previous version"
                    } catch (Exception e) {
                        echo "❌ Rollback failed: ${e.message}"
                    }
                }
            }
        }
        unstable {
            script {
                echo "⚠️ Deployment UNSTABLE for ${SERVICE_NAME}"
                echo "Build: ${env.BUILD_NUMBER}"
                echo "Status: UNSTABLE - Review health checks"
            }
        }
        always {
            script {
                echo "🧹 Cleaning up workspace..."
                try {
                    cleanWs()
                } catch (Exception e) {
                    echo "⚠️ Workspace cleanup skipped (pod already terminated)"
                }
            }
        }
    }
}
