#!/usr/bin/env groovy
/**
  This is the Jenkinsfile for the Logstash Deployer targeting the
  CI normalizer OpenShift project
 */
properties(
  [
    [$class: 'BuildConfigProjectProperty', name: '', namespace: '', resourceVersion: '', uid: ''],
    buildDiscarder(logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '10', daysToKeepStr: '', numToKeepStr: '10')),
    [$class: 'HudsonNotificationProperty', enabled: false],
    [$class: 'RebuildSettings', autoRebuild: false, rebuildDisabled: false],
    parameters(
      [
        string(defaultValue: 'external_config', description: 'commit ref hash, branch, or tag to build', name: 'GIT_REF'),
        string(defaultValue: 'dh-prod-ingest', description: 'OpenShift Project to deploy  into', name: 'OPENSHIFT_PROJECT'),
        string(defaultVaule: 'two-kafka.dh-two-message-bus.svc.cluster.local:9092', description: 'Destination Kafka cluster', name: 'KAFKA_BROKER')
      ]
    ),
    pipelineTriggers([])
  ]
)

ansiColor('xterm') {
  timestamps {
    node('dhslave') {
      configFileProvider([configFile( fileId: 'kubeconfig', variable: 'KUBECONFIG')]){
        wrap([$class: 'MaskPasswordsBuildWrapper', varMaskRegexes: [[regex: '\\(item=\\{\'key\'.*\'value\'.*$']]]) {
          stage('Checkout SCMs') {
            checkout_scms()
          }
          stage('Trigger Deployment Playbook') {
            run_deployment()
          }
        }
      }
    }
  }
}

def checkout_scms() {
    checkout poll: false, scm: [
      $class: 'GitSCM',
      branches: [[name: "${GIT_REF}"]],
      doGenerateSubmoduleConfigurations: false,
      extensions: [
        [$class: 'WipeWorkspace'],
        [$class: 'RelativeTargetDirectory', relativeTargetDir: 'container-rsyslog']
      ],
      submoduleCfg: [],
      userRemoteConfigs: [[url: 'https://github.com/t0ffel/docker-rsyslog']]
    ]
}

def run_deployment() {
  try {
      sh '''

# Run deployment script
cd $WORKSPACE/container-rsyslog/openshift
NAMESPACE=${OPENSHIFT_PROJECT} ./deploy-normalizer.sh ${KAFKA_BROKER}
'''
  } catch (err) {
    echo 'Exception caught, being re-thrown...'
    throw err
  }
}