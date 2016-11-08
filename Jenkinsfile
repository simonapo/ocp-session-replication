// Responsible for
//  1. Determining new version of a build
//  2. building application
//  3. executing Junit tests included in the app

node {
    stage 'Build'
    //build 'session-replication'

    // Publish to a QA environment
    stage 'QA'
    echo "PROJECT = $PROJECT"
    openshiftBuild apiURL: '', authToken: '', bldCfg: 'webapp-qa', checkForTriggeredDeployments: 'false', namespace: PROJECT, verbose: 'false'
    openshiftVerifyBuild apiURL: '', authToken: '', bldCfg: 'webapp-qa', checkForTriggeredDeployments: 'false', namespace: PROJECT, verbose: 'false'

    // Wait until authorization to push to production
    stage 'Approve'
    timeout(time: 2, unit: 'DAYS') {
        input message: 'Do you want to deploy into production?'
    }

    // Push to production
    // Uses `oc` utility of OSE to trigger S2I BuildConfig object in prod environment
    stage 'Production'
    echo "PROJECT = $PROJECT"
    openshiftBuild apiURL: '', authToken: '', bldCfg: 'webapp-prod', checkForTriggeredDeployments: 'false', namespace: PROJECT, verbose: 'false'
    openshiftVerifyBuild apiURL: '', authToken: '', bldCfg: 'webapp-prod', checkForTriggeredDeployments: 'false', namespace: PROJECT, verbose: 'false'
}
