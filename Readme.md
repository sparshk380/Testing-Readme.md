
# Configure Kubernetes Pod as a Slave for Jenkins Master running in Kubernetes Cluster.


### Pre-requisites:
 - Jenkins Master running on Kubernetes Cluster 

## Steps: 

### Step 1: Install the necessary plugins

- Navigate to Jenkins Dashboard in a web browser
- Go to manage Jenkins > Manage plugins
- Click on Available tab and search for Kubernetes Plugin

### Step 2: Configure Jenkins to connect to Kubernetes

1. Add Kubernetes Cloud Configuration.

- Go to Manage Jenkins > Manage Nodes > Configure Clouds

![Screenshot](<Jenkins-1.jpg>)


- Click on Add a new cloud and select Kubernetes.

![Screenshot](<Jenkins-2.jpg>)

2. Configure Kubernetes Cloud Details.

- Provide a name for the Kubernetes cloud (e.g., Kubernetes Cloud).
- Set the Kubernetes URL (https://kubernetes.default if Jenkins is running within  the cluster). (Leave it blank if the Jenkins master and slave are both inside the same cluster as in this case.)
- Provide the Jenkins URL (e.g., http://<jenkins-service-name>:8080).
- If necessary, add Kubernetes credentials:
    * Click on Add next to Credentials and select Kubernetes Service Account.
    * Use the default service account or create a dedicated one with appropriate permissions.


![Screenshot](<3.jpg>)
 
3. Configure Pod Templates:
- Click on Add Pod Template to define a pod template.
- Set a name for the pod template (e.g., jnlp-agent).
- Define the Docker image for the Jenkins agent (e.g., jenkins/inbound-agent).
- Specify container details (name, image, resource limits, etc.).

![Screenshot](<4.jpg>)

![Screenshot](<5.jpg>)

![Screenshot](<6.jpg>)

![Screenshot](<Screenshot 2024-07-26 161620.png>)

### Step 3: Create a Jenkins Pipeline:

1. Create a pipeline Job:
  - Go to Jenkins dashboard > New Item > Pipeline.
  - Provide a name for your pipeline and select Pipeline.

2. Define the pipeline Script:
 - In the pipeline configuration, use the following script to leverage Kubernetes Agents: 

 ```
 pipeline {
    agent {
        kubernetes {
            label 'k8s-agent'
            yaml '''
            apiVersion: v1
            kind: Pod
            metadata:
              labels:
                some-label: some-label-value
            spec:
              containers:
              - name: jnlp
                image: jenkins/inbound-agent
                args: ['$(JENKINS_SECRET)', '$(JENKINS_NAME)']
              - name: docker
                image: docker:latest
                command:
                - cat
                tty: true
            '''
        }
    }
    stages {
        stage('Test') {
            steps {
                sh 'echo "Running tests"'
            }
        }
    }
}

```

### Step 4: Test the setup

1. Trigger the pipeline
- Manually trigger the pipeline job from the Jenkins Dashboard.
- Ensure that the pipeline creates a Kubernetes pod, executes the stages, and deletes the pod after the execution completes.








