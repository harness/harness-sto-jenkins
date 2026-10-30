# Harness STO Jenkins

Jenkins runner for Harness Security Testing Orchestration (STO). The onboarding wizard clones a release tag of this repo and loads `lib/StoScan.groovy`.

Pin a release tag. Do not clone `main`.

The Jenkins agent needs Docker. Scanner images are `harness/<scanner>-job-runner`. When `dockerImageTag` is omitted, the tag is `latest`.

## Usage

Set the Harness account values on the pipeline. `HARNESS_TOKEN` is a Jenkins Secret text credential. Scanner tokens stay in that stage's `withCredentials` block (`SCANNER_ACCESS_ID`, `SCANNER_ACCESS_TOKEN`, `SCANNER_DOMAIN`). This repo does not contain them.

```groovy
environment {
    HARNESS_DOMAIN     = 'https://app.harness.io'
    HARNESS_ACCOUNT_ID = 'your-account-id'
    HARNESS_ORG_ID     = 'your-org-id'
    HARNESS_PROJECT_ID = 'your-project-id'
    HARNESS_TOKEN      = credentials('harness-token')
}

stages {
    stage('Checkout STO library') {
        steps {
            dir('sto-jenkins-lib') {
                deleteDir()
                sh '''
                    git clone --depth 1 --branch <release-tag> \
                      https://github.com/harness/harness-sto-jenkins.git .
                '''
            }
        }
    }

    stage('Bandit') {
        steps {
            script {
                def sto = load 'sto-jenkins-lib/lib/StoScan.groovy'
                sto.init(this)
                sto.run([
                    scanner    : 'bandit',
                    scanMode   : 'orchestration',
                    scanConfig : 'default',
                    targetType : 'repository',
                ])
            }
        }
    }
}
```

A repository scan mounts the Jenkins workspace at `/harness`. Set `sourceRoot` when the code is not the workspace root. Set `workspace` when the scan should run in a subdirectory of that checkout.
