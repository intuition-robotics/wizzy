@Library('ir-jenkins-pipeline-libs')
import ir.jenkins.utils.*
import ir.jenkins.ci.Constants


node("ci"){

  timestamps(){

    String stageName = ""
    try{
          stage('Setup') {
              stageName = env.STAGE_NAME
              echo "### Cleanup workspace"
              deleteDir()
              currentBuild.description = "Env: <b>${env.ENVIRONMENT}</b>"
          }

          stage ("Checkout wizzy") {
              stageName = env.STAGE_NAME
              echo "### Checkout wizzy"
              GitUtils.checkout(this, Constants.ORG_NAME, "wizzy", "master" , "")
          }

          stage ("Build wizzy") {
              stageName = env.STAGE_NAME
              echo "### Build wizzy latest docker image"
              sh "docker build -t wizzy ."

          }

          stage ("Backup Grafana Dashboards") {
              stageName = env.STAGE_NAME
              echo "### Backup Grafana Dashboards"

              String grafanaUrl = (env.ENVIRONMENT == "dev") ? Constants.GRAFANA_URL_DEV : Constants.GRAFANA_URL_PROD

              withCredentials([string(credentialsId: "grafana-admin-pass-${env.ENVIRONMENT}", variable: "grafanaAdminPass")]) {
                sh """
                docker run --name wizzy --rm --entrypoint /bin/ash -v ${env.WORKSPACE}/dashboards:/app/dashboards wizzy \
                      -c "wizzy init && \
                      wizzy set grafana url ${grafanaUrl} && \
                      wizzy set grafana username admin && \
                      wizzy set grafana password ${grafanaAdminPass} && \
                      wizzy import dashboards"
                """
              }

              echo "Archive exported dashboards"
              sh "tar cvfz dashboards.tar.gz dashboards"
              archiveArtifacts artifacts: "dashboards.tar.gz", excludes: ""

          }

        } catch (Exception e) {
            currentBuild.result = "FAILURE"
            stage("Notify") {
                echo "### Notify"
                def notify = new Notify(this)
                notify.slackNotify("#infrastructure_alerts", "slack_jenkins_ci", "", "", "", false, "Failed to export dashboards from ${env.ENVIRONMENT}!", false, stageName)
            }
            throw e
        }
    }

}
