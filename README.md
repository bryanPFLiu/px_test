# performance-plugin-test

## Issues

standardResults.xml generate by Jenkins performance plugin is not hanlding double-byte characters well after jenkins migration.
There is no such a problem before Jenkins migration.

Jenkins and plugin versions:
- Jenkins version: 2.303.1
- Performance plugin: 3.17 ~ 3.20

Tested env:
- Jenkins docker image
- Linux server on CentOS 7

I tried to run a test which is similar to hudson.plugins.performance.PerformancePublisherTest.java from my local IDE but unable to recreate the problem, the generated xml contains correct double-byte characters.
So, probably the problem was caused when performing artifacts serialization or something else?

## steps to recreate

1. clone jenkins docker image

```sh
docker run -p 8080:8080 --name=jenkins-master -d jenkins/jenkins:2.303.1
```

2. install suggested plugins and performance plugin

3. restart the server

4. create a pipeline job as below

``` yaml
#!groovy

pipeline {
    agent {
        label "master"
    }
    
    stages {
        stage('SCM Checkout') {
            steps {
                script {
                    sh 'rm -rf ./*'
                    git branch: 'master',
                      url: 'https://github.com/bryanPFLiu/px_test.git'
                    sh 'git clean -fdx'
                }
            }
        }//stage
        stage('Load Test') {
            steps {
                sh "env"
                sh "java -version"
            }//steps
        }//stage
        stage('Generate Performance Report') {
            steps {
                perfReport(
                        compareBuildPrevious: true,
                        errorFailedThreshold: 0,
                        errorUnstableThreshold: 0,
                        filterRegex: '',
                        nthBuildNumber: 1,
                        relativeFailedThresholdNegative: 1.0,
                        relativeFailedThresholdPositive: 1.0,
                        relativeUnstableThresholdNegative: 1.0,
                        relativeUnstableThresholdPositive: 1.0,
                        sourceDataFiles: 'load/report/*.jtl'
                )
                copyArtifacts(
                        projectName: "${JOB_NAME}",
                        filter: "standardResults.xml",
                        target: "./load",
                        selector: specific("${BUILD_NUMBER}")
                )
            } // steps
        } // stage
        
    } // stages
} // 
```

5. run the pipeline and check standardResults.xml in Build Artifacts from Jenkins web page

6. the double-byte characters are showing ?? in the result xml file
