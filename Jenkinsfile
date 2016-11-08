#!groovy

node {
    stage 'Build'
        //build 'session-replication'

    openshiftBuild apiURL: '', authToken: '', bldCfg: 'webapp-dev', checkForTriggeredDeployments: 'false', namespace: OCP_PROJECT, verbose: 'false'
    openshiftVerifyBuild apiURL: '', authToken: '', bldCfg: 'webapp-dev', checkForTriggeredDeployments: 'false', namespace: OCP_PROJECT, verbose: 'false'

    stage 'Deploy to Dev'
    openshiftDeploy apiURL: '', authToken: '', depCfg: 'webapp-qa', namespace: OCP_PROJECT, verbose: 'false', waitTime: ''
    openshiftVerifyDeployment apiURL: '', authToken: '', depCfg: 'webapp-qa', namespace: OCP_PROJECT, replicaCount: '1', verbose: 'false', verifyReplicaCount: 'false', waitTime: ''

    openshiftTag alias: 'false', apiURL: '', authToken: '', destStream: 'webapp', destTag: 'qa', destinationAuthToken: '', destinationNamespace: OCP_PROJECT, namespace: OCP_PROJECT, srcStream: 'webapp', srcTag: 'latest', verbose: 'false'

    timeout(time: 2, unit: 'DAYS') {
        input message: 'Do you want to deploy into Q&A?'
    }

    // Publish to a QA environment
    stage 'Deploy to QA'
    openshiftDeploy apiURL: '', authToken: '', depCfg: 'webapp-qa', namespace: OCP_PROJECT, verbose: 'false', waitTime: ''
    openshiftVerifyDeployment apiURL: '', authToken: '', depCfg: 'webapp-qa', namespace: OCP_PROJECT, replicaCount: '1', verbose: 'false', verifyReplicaCount: 'false', waitTime: ''

    openshiftTag alias: 'false', apiURL: '', authToken: '', destStream: 'webapp', destTag: 'prod', destinationAuthToken: '', destinationNamespace: OCP_PROJECT, namespace: OCP_PROJECT, srcStream: 'webapp', srcTag: 'qa', verbose: 'false'

    // Wait until authorization to push to production
    stage 'Approve'
    timeout(time: 2, unit: 'DAYS') {
        input message: 'Do you want to deploy into production?'
    }

    // Push to production
    // Uses `oc` utility of OSE to trigger S2I BuildConfig object in prod environment
    stage 'Deploy to Production'
    openshiftDeploy apiURL: '', authToken: '', depCfg: 'webapp-prod', namespace: OCP_PROJECT, verbose: 'false', waitTime: ''
    openshiftVerifyDeployment apiURL: '', authToken: '', depCfg: 'webapp-prod', namespace: OCP_PROJECT, replicaCount: '1', verbose: 'false', verifyReplicaCount: 'false', waitTime: ''
}
