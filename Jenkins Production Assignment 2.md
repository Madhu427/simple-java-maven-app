                                                          Jenkins Production Assignment 2


<img width="523" height="199" alt="image" src="https://github.com/user-attachments/assets/710411e0-225a-4894-a6e7-e67e1ea37a57" />


##############################################################
pipeline {
    agent { label 'java-build-node' } // Scalability/Distributed requirement

    tools {
        maven 'M3' // Matches the name you set in Global Tool Configuration
    }

    options {
        retry(2) // Graceful failure management
        timeout(time: 15, unit: 'MINUTES')
    }

    environment {
        // Simulating a persistent cache location for Maven dependencies
        M2_CACHE = "/var/lib/jenkins/maven-cache" 
        PROD_CRED = credentials('prod-api-key') // Security requirement
    }

    stages {
        stage('Checkout') {
            steps {
                // Pulls code from your specific URL
                git 'https://github.com/Madhu427/simple-java-maven-app.git'
            }
        }

        stage('Build') {
            steps {
                echo "Building with Maven Caching..."
                // Use -Dmaven.repo.local to simulate logical caching of dependencies
                sh "mvn -Dmaven.repo.local=${M2_CACHE} clean compile"
            }
        }

        stage('Test') {
            steps {
                echo "Running Unit Tests..."
                sh "mvn -Dmaven.repo.local=${M2_CACHE} test"
            }
        }

        stage('Package') {
            steps {
                sh "mvn -Dmaven.repo.local=${M2_CACHE} package -DskipTests"
                // Archive the resulting JAR file
                archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
            }
        }

        stage('Deploy (Simulated)') {
            steps {
                echo "Deploying JAR to production using ${PROD_CRED}..."
                // Logic check: ensure the JAR exists before "deploying"
                sh '[ -f target/*.jar ] && echo "Deployment Success" || exit 1'
            }
        }
    }

    post {
        failure {
            echo "Build failed. Alerting DevOps Team..."
        }
    }
}
#########################################################################

<img width="530" height="378" alt="image" src="https://github.com/user-attachments/assets/95ce7a46-115c-4a72-a833-c990d2947fa8" />

#################################################################
<img width="580" height="323" alt="image" src="https://github.com/user-attachments/assets/d3697433-adb0-47ad-9a6e-4b44a2786449" />



Step 1: Install and Configure Jenkins from Scratch
Since Docker is not allowed, we will perform a native installation.

Install Java (Prerequisite): Jenkins is a Java application. Install OpenJDK 17.

Bash

sudo apt update
sudo apt install fontconfig openjdk-17-jre -y
Add Jenkins Repository:

Bash

sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/" | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
Install Jenkins:

Bash

sudo apt update
sudo apt install jenkins -y
Start and Enable Service:

Bash

sudo systemctl start jenkins
sudo systemctl enable jenkins
Step 2: Install and Manage Plugins via GUI
After accessing Jenkins for the first time at http://your-server-ip:8080, you need to manage your toolset.

Unlock Jenkins: Retrieve the initial password.

Bash

sudo cat /var/lib/jenkins/secrets/initialAdminPassword
Plugin Installation:

During the initial wizard, select "Select plugins to install."

Ensure Git and Pipeline are checked.

To manage later: Go to Manage Jenkins > Plugins > Available plugins to search for any missing tools.

Step 3: Secure Jenkins & Create Admin User
To restrict anonymous access and secure the environment:

Create First Admin: Complete the setup wizard by creating a username (e.g., jenkins_admin) and a strong password.

Verify Security Realm:

Go to Manage Jenkins > Security.

Ensure Security Realm is set to "Jenkins’ own user database."

Ensure Authorization is set to "Logged-in users can do anything" (temporarily, until Step 4).

Uncheck "Allow anonymous read access" to ensure only authorized users can see the dashboard.

Step 4: Enforce Role-Based Authorization
Standard production environments use the Role-based Authorization Strategy plugin.

Install Plugin: Go to Plugins and install "Role-based Authorization Strategy."

Change Strategy: Go to Manage Jenkins > Security > Authorization and select Role-Based Strategy. Save changes.

Manage Roles: Go to Manage Jenkins > Manage and Assign Roles > Manage Roles.

Admin Role: Give it all permissions.

Developer Role: Give it permissions like Overall:Read, Job:Build, Job:Read, and Job:Workspace.

Assign Roles: Go to Assign Roles and map your created Jenkins users to these specific roles.

Step 5: Enable Update Center and Verify Compatibility
Maintaining plugin health is critical for "production-ready" systems.

Configure Update Center:

Go to Manage Jenkins > Plugins > Advanced settings.

Verify the Update Site URL is correct (usually https://updates.jenkins.io/update-center.json).

Check for Updates: Go to the Updates tab in the Plugin Manager to see available security patches or newer versions.

Verify Compatibility: Before clicking "Install," check the "Compatibility" column. Jenkins will flag plugins that require a newer core version or have known security vulnerabilities.


<img width="667" height="301" alt="image" src="https://github.com/user-attachments/assets/21e8794c-47f4-4626-b871-f63eecfdba40" />


<img width="544" height="443" alt="image" src="https://github.com/user-attachments/assets/39897cf1-8a09-4320-8b6b-ecbd6e383e6c" />

#################################################

<img width="566" height="222" alt="image" src="https://github.com/user-attachments/assets/d3d69d52-cfa9-4fa1-8c97-03c4fcaef56c" />



Follow these steps for both required jobs: build-and-test-service-A and build-and-test-service-B.

Create Jenkins Jobs:

Select New Item on the dashboard, enter the specific job name (e.g., build-and-test-service-A), and select  "Pipeline".

Configure Git Branching:

In the Source Code Management section, provide your Git repository URL.

Crucially, configure each job to clone from a different branch (e.g., Job A clones the dev branch, while Job B clones the feature branch).

Simulate Unit Tests:

Add an Execute Shell build step.

Run basic shell script logic such as echo "Running unit tests..." followed by a conditional check to simulate a testing environment.

Generate and Save Build Artifacts:

In the same shell script, include a command to save build artifacts by creating a compressed file, such as tar -czvf service-a-build.tar.gz ./*.

Ensure these fake .tar.gz files are generated directly within the Jenkins workspace.

Archive Build Artifacts:

Add a Post-build Action titled Archive the artifacts.

Specify the file pattern (e.g., *.tar.gz) to ensure the build output is preserved and accessible from the Jenkins job dashboard

Jenkinsfile
---------------------------------------
pipeline {
    agent any

    stages {
        stage('Checkout Source') {
            steps {
                // Task: Source Code Management with Git
                // For Job A use 'dev', for Job B use 'feature'
                git branch: 'feature', url: 'https://github.com/Madhu427/simple-java-maven-app.git'
            }
        }

        stage('Build and Simulate Tests') {
            steps {
                // Task: Run basic shell script logic to simulate unit tests
                sh '''
                    #!/bin/bash
                    echo "Starting Build and Test Process..."
                    
                    # Simulate Test Result
                    TEST_RESULT=0 
                    
                    if [ $TEST_RESULT -eq 0 ]; then
                        echo "Unit Tests Passed Successfully!"
                    else
                        echo "Unit Tests Failed!"
                        exit 1
                    fi

                    # Task: Save build artifacts (fake .tar.gz files) in workspace
                    echo "Generating Build Version: 1.0.$BUILD_NUMBER" > build_info.txt
                    tar -czvf service-build-${BUILD_NUMBER}.tar.gz build_info.txt
                '''
            }
        }
    }

    post {
        always {
            // Task: Archive build artifacts
            // This ensures *.tar.gz files are saved and visible on the job page
            archiveArtifacts artifacts: '*.tar.gz', fingerprint: true
        }
    }
}
---------------------------------------
<img width="596" height="419" alt="image" src="https://github.com/user-attachments/assets/9e01a1c2-7975-4e80-86c9-9caf017015d4" />


<img width="653" height="443" alt="image" src="https://github.com/user-attachments/assets/d7be20d0-2236-4f97-8954-d995f871d6ae" />

<img width="713" height="240" alt="image" src="https://github.com/user-attachments/assets/4036b38f-72c5-443e-ba26-5cd5df464d42" />

JOB B

<img width="603" height="410" alt="image" src="https://github.com/user-attachments/assets/ad44faeb-e817-4caf-b27b-8cb5f4fbe1fc" />

<img width="546" height="463" alt="image" src="https://github.com/user-attachments/assets/4bc776f8-43cc-4bf3-b73f-581289ca6842" />

<img width="515" height="260" alt="image" src="https://github.com/user-attachments/assets/62d2100e-23d6-449c-8c8b-2d6060fa1c32" />


######################################################################################

<img width="554" height="143" alt="image" src="https://github.com/user-attachments/assets/1e19fe2f-de6c-4b68-af96-3a495415cebd" />


----------------------------------------------------------------------
pipeline {
    agent any
    
    // Task 3: Use parameterized jobs for dynamic selection
    parameters {
        string(name: 'BRANCH_NAME', defaultValue: 'master', description: 'Branch selection (e.g., master or feature)')
        booleanParam(name: 'DEBUG_MODE', defaultValue: false, description: 'Debug mode toggle')
        choice(name: 'ENVIRONMENT', choices: ['dev', 'staging', 'prod'], description: 'Simulated environment selection')
    }

    stages {
        stage('Checkout Source') {
            steps {
                // Task: Source Code Management with Git
                // Uses the BRANCH_NAME parameter for dynamic cloning
                git branch: "${params.BRANCH_NAME}", url: 'https://github.com/Madhu427/simple-java-maven-app.git'
            }
        }

        stage('Build and Simulate Tests') {
            steps {
                // Task: Run basic shell script logic to simulate unit tests
                sh '''
                    #!/bin/bash
                    echo "Starting Build on Environment: ${ENVIRONMENT}"
                    
                    # Check Debug Mode toggle
                    if [ "${DEBUG_MODE}" = "true" ]; then
                        echo "DEBUG: System path is $PATH"
                        set -x # Enable verbose logging
                    fi

                    # Simulate Test Result
                    TEST_RESULT=0 
                    
                    if [ $TEST_RESULT -eq 0 ]; then
                        echo "Unit Tests Passed Successfully!"
                    else
                        echo "Unit Tests Failed!"
                        exit 1
                    fi

                    # Task: Save build artifacts (fake .tar.gz files) in workspace
                    echo "Generating Build Version: 1.0.$BUILD_NUMBER" > build_info.txt
                    echo "Environment: ${ENVIRONMENT}" >> build_info.txt
                    tar -czvf service-build-${BUILD_NUMBER}.tar.gz build_info.txt
                '''
            }
        }
    }

    post {
        always {
            // Task: Archive build artifacts
            // This ensures *.tar.gz files are saved and visible on the job page
            archiveArtifacts artifacts: '*.tar.gz', fingerprint: true
        }
    }
}

--------------------------------------------------

<img width="874" height="349" alt="image" src="https://github.com/user-attachments/assets/4e81a0b5-47ba-469b-8158-0d7c4b2aa825" />

<img width="704" height="455" alt="image" src="https://github.com/user-attachments/assets/6d45b8dc-83a5-42a1-a031-027798b43423" />

<img width="617" height="297" alt="image" src="https://github.com/user-attachments/assets/41e3d0bf-3d00-462a-b7b5-a55c49541a3c" />


#############################################################################################


<img width="515" height="131" alt="image" src="https://github.com/user-attachments/assets/6fb8ae9b-6f7a-44e3-a2d9-b3062f3e95fd" />


------------------------------------------------------------------------
pipeline {
    agent any
    
    stages {
        stage ('BUILD') {
            steps {
                sh '''echo "DEPLOYMENT STATUS: Starting deployment to ${ENVIRONMENT} environment..."
echo "Deploying Service A and Service B as a unified release."
echo "Release Version: 1.0.${BUILD_NUMBER}" > deployment_manifest.txt
echo "Includes Service A Build: ${UPSTREAM_BUILD_NUMBER_SERVICE_A}" >> deployment_manifest.txt
echo "Includes Service B Build: ${UPSTREAM_BUILD_NUMBER_SERVICE_B}" >> deployment_manifest.txt'''
            }
        }
    }
}


-------------------------------------



<img width="947" height="394" alt="image" src="https://github.com/user-attachments/assets/7ad7e64f-291c-415b-b620-aba4d09c5ab2" />

########################################################################

<img width="511" height="224" alt="image" src="https://github.com/user-attachments/assets/42383a1d-4f2b-42ab-8bec-dba60ffd91a5" />

1. Environment Variables (environment)
We define APP_NAME and CACHE_DIR globally. This makes the script maintainable; if your cache location changes, you only update it in one place.

2. Simulated Caching
We use a shell script to check if a directory exists.

Logic: If the directory exists, we "reuse" dependencies.

Logic: If not, we "fetch" them (simulate by creating a text file). This satisfies the requirement to use conditional logic instead of fetching every time.

3. Retry Logic (retry)
The retry(2) block is wrapped around the test stage. If the shell script exits with an error (exit 1), Jenkins will automatically restart that specific stage up to two additional times before marking the whole build as failed.

4. Conditional Execution (when)
This is a production safety feature. The when directive ensures that the "Build & Package" stage only runs on specific branches. If someone runs this pipeline on a temporary feature branch, Jenkins will skip the packaging stage to save resources.

5. Post Conditions (post)
success: Only triggers if every stage finishes without error. This is where we archive our .tar.gz artifacts so they appear on the Jenkins dashboard.

failure: Useful for sending alerts or cleaning up the workspace if the build crashes.
---------------------------------------
pipeline {
    agent any

    environment {
        // Task: Use of environment variables
        APP_NAME = "MultiService-App"
        CACHE_DIR = "maven_cache"
    }

    stages {
        stage('Initialize & Cache') {
            steps {
                // Task: Simulate caching
                sh '''
                    if [ -d "${CACHE_DIR}" ]; then
                        echo "Dependencies found in cache. Skipping download."
                    else
                        echo "Cache empty. Downloading dependencies..."
                        mkdir ${CACHE_DIR}
                        echo "dependency-v1.0" > ${CACHE_DIR}/lib.txt
                    fi
                '''
            }
        }

        stage('Checkout Source') {
            steps {
                // Task: Proper use of stages and steps
                git branch: 'main', url: 'https://github.com/Madhu427/simple-java-maven-app.git'
            }
        }

        stage('Simulate Tests') {
            // Task: Implement retry logic for test stages (max 2 retries)
            retry(2) {
                steps {
                    sh '''
                        echo "Running Unit Tests..."
                        # Simulate a 10% chance of failure to test retry logic
                        if [ $(( $RANDOM % 10 )) -eq 0 ]; then
                            echo "Random network failure detected!"
                            exit 1
                        fi
                        echo "Tests Passed!"
                    '''
                }
            }
        }

        stage('Build & Package') {
            // Task: Use of when directive to conditionally run stages
            // Only runs if we are on the 'main' or 'prod' branch
            when {
                anyOf {
                    branch 'main'; branch 'prod'
                }
            }
            steps {
                sh '''
                    echo "Packaging ${APP_NAME}..."
                    tar -czvf service-build-${BUILD_NUMBER}.tar.gz ./*
                '''
            }
        }
    }

    post {
        // Task: Proper use of post conditions
        success {
            echo "Build and Test successful! Ready for Deployment."
            archiveArtifacts artifacts: '*.tar.gz', fingerprint: true
        }
        failure {
            echo "Build failed. Check the logs for caching or test errors."
        }
    }
}
-----------------------------------------


<img width="651" height="410" alt="image" src="https://github.com/user-attachments/assets/9cbb9316-3b6d-4946-bedc-414123ec2e6a" />


<img width="514" height="239" alt="image" src="https://github.com/user-attachments/assets/4d6d675a-9d33-46c0-853d-67ffdc118ff2" />

########################################################

<img width="419" height="203" alt="image" src="https://github.com/user-attachments/assets/ef515f21-164e-4b4e-ac6a-72fe9008ddb5" />



<img width="786" height="70" alt="image" src="https://github.com/user-attachments/assets/57bdc5b7-7ea6-45e4-bfef-4263c66e3240" />

<img width="554" height="275" alt="image" src="https://github.com/user-attachments/assets/ec7e4816-139c-4674-a1ea-c9d28a18ab8c" />

<img width="682" height="311" alt="image" src="https://github.com/user-attachments/assets/518dbf33-12e6-41cc-b5d7-b6bf5e26c007" />

---------------------
pipeline {
    // We set agent to 'none' at the top level so we can specify agents for each stage
    agent none 

    environment {
        APP_NAME = "Simple-Java-App"
    }

    stages {
        stage('Build and Test') {
            // Task: Assign build jobs to specific agent
            agent { label 'build-worker' } 
            steps {
                git branch: 'main', url: 'https://github.com/Madhu427/simple-java-maven-app.git'
                
                // Task: Simulate unit tests with retry logic
                retry(2) {
                    sh '''
                        echo "Running tests on BUILD-WORKER node..."
                        TEST_RESULT=0
                        if [ $TEST_RESULT -eq 0 ]; then
                            echo "Tests Passed!"
                        else
                            exit 1
                        fi
                    '''
                }
                
                // Task: Save and Archive build artifacts
                sh 'tar -czvf app-build-${BUILD_NUMBER}.tar.gz ./*'
                archiveArtifacts artifacts: '*.tar.gz', fingerprint: true
            }
        }

        stage('Deploy to Production') {
            // Task: Assign different jobs/stages to different agents
            agent { label 'production-slave' }
            // Task: Use 'when' directive to ensure deployment only happens on main
            when { branch 'main' } 
            steps {
                sh '''
                    echo "Deploying to PRODUCTION node..."
                    echo "Unpacking artifacts and simulating deployment..."
                    # In a real scenario, you would copy the artifact from the master/build node
                    echo "Deployment to Production Salve Successful!"
                '''
            }
        }
    }

    post {
        always {
            echo "Pipeline finished. Check console logs for 'Building remotely on...' to verify nodes."
        }
    }
}
-------------------------

<img width="818" height="515" alt="image" src="https://github.com/user-attachments/assets/53ca4062-b637-4251-ad6b-226f963759e6" />

<img width="661" height="386" alt="image" src="https://github.com/user-attachments/assets/e28a8df1-dd9c-49e7-b185-2a327bdb523b" />














