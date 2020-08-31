# sample.edge-mnist-flows

The sample.edge-mnist is a sample that demonstrates IBM’s Cloud Pak for Data (CP4D) Edge Analytics on the popular MNIST dataset. 

The sample consists of 2 main parts:

<img src="https://github.com/IBMStreams/sample.edge-mnist-flows/blob/main/screenshots/arch.png" width="720"> 

1. The Streams Flows - An IBM Streams app (developed using Streams Flows) running on remote edge systems, where real-time data from IOT devices (or in our case the static MNIST dataset) is processed and scored, with metrics being sent back to the CP4D hub for further analysis. For an overview of Streams Flows, click [here](https://www.youtube.com/watch?v=rVTOnt0nbDA) 

2. The Notebook -  A python Jupyter notebook that creates an IBM Streams app on Cloud Pak for Data (CP4D) to receive data from the remote edge systems (through Apache Kafka) for further processing of the data.
	
## Requirements 
- IBM Cloud Pak for Data (CP4D) Cluster v3.0.1 with [Streams / Streams Flows](https://www.ibm.com/support/producthub/icpdata/docs/content/SSQNUZ_current/svc-welcome/streams.html) and [Watson Machine Learning](https://www.ibm.com/support/producthub/icpdata/docs/content/SSQNUZ_current/svc-welcome/wml.html) installed 
- [CP4D Edge Analytics beta](https://www.ibm.com/support/knowledgecenter/SSQNUZ_3.0.1/svc-welcome/edge.html)
- [CP4D Streams instance](https://www.ibm.com/support/producthub/icpdata/docs/content/SSQNUZ_current/cpd/svc/streams/provision.html)
- Kafka / [IBM Event Streams](https://www.ibm.com/cloud/event-streams) implementation with at least 1 topic
- Familiarity with Python, IBM Streams, WML and Kafka/IBM Event Streams

## Instructions 
To get started with building the edge-mnist sample application, clone this repo and follow the steps below

#### 1. Set up CP4D Project 
- Create a CP4D project with git integration and Jupyterlab IDE
  - On the CP4D new project page, create a new empty project, choose a name and check the git integration box underneath
  - On Github, create a [personal access token](https://docs.github.com/en/github/authenticating-to-github/creating-a-personal-access-token) with repo access (first checkbox) and copy the token
  - On the new project page, click 'New Token', paste the access token, give it a name, and finally click "Continue"
  - Back on Github, create an empty Git repo (e.g. mnist) and copy the HTTPS url for the repo
  - Paste the URL in the Repository URL field, select the branch, check the box labeled "Edit notebooks only with the JupyterLab IDE", and finally click "Create"
  - Once the project is created, open the project
  - Launch the JupyterLab IDE by clicking the "Launch IDE" dropdown and select 'JupyterLab'
  - For more information, please click [here](https://www.ibm.com/support/producthub/icpdata/docs/content/SSQNUZ_current/wsj/getting-started/projects.html)
 
- Upload files to Jupyterlab IDE
  - After launching Jupyterlab, there should be 2 folders in the left panel - `project_data_assets` and `project_git_repo`
  - Select `project_git_repo`, then the folder with your github repo name, then `assets` and finally `jupyterlab`
  - At this point, you should be in the `project_git_repo/<github-repo-name>/assets/jupyterlab/` directory
  - Upload the following files to the current directory:
    - `Build-metro-app.ipynb`
    - `digit-support.py`
    - `metrorender.py`
    - `render-metro-view.ipynb`
    - `model_upload.ipynb`
    - `sklearn-SVC-model`
 
 - Upload model & create deployment
   - While still in Jupyterlab, open and follow the instructions in the script `model_upload.ipynb` to populate it with the required details. Finally run it to upload the model
   - Go back to your project and under the Settings tab, click "Associate a deployment space"
   - Create a new deployment space by entering a name and clicking "Associate"
   - Click on the "Assets" tab of the project and under "Models" section, click "Promote" of the sklearn-SVC-model options and finally click "Promote to space"
   - Navigate to your deployment space, and click the "Deploy" button of the sklearn-SVC-model, select "Online" for the deployment type, enter a name and click 'Create'
   - For more information, please click [here](https://www.ibm.com/support/producthub/icpdata/docs/content/SSQNUZ_current/wsj/analyze-data/ml-spaces_local.html )


#### 2. Streams Flows app
- Upload flow app to CP4D (the `sample.edge-mnist.stp` file)
  - Exit out of Jupyterlab IDE and go back to your project
  - Click the assets tab, select the blue "Add to project" button, select Streams Flows, from file. 
  - From there, choose a name, select an existing Streams instance, then click create
  - For more information, please click [here](https://www.ibm.com/support/producthub/icpdata/docs/content/SSQNUZ_current/wsj/streaming-pipelines/creating-pipeline-import.html)
  
- Complete Streams Flows app 
  - To complete the Streams Flow app, we need to connect the app to the model deployment created earlier. 
  - The WML node connects to the WML service, retrieves the model we uploaded earlier, and uses it to score the MNIST dataset images
  - With the Streams Flow open, click "Edit the streams flow"
  - To connect the app to the WML deployment, select the WML MNIST model node, and in the settings panel on the right, change the model to the model deployment created earlier under the WML MODELS dropdown.
  - Make sure the "Inline Scoring" checkbox under Runtime is ticked. This tells WML to download the model to do scoring on-device (in this case, the remote edge systems)
  - Configure the output
    - In the same settings panel, under "Schema", click the "Edit" button, click "Reset schema"
    - Click the "Add attributes from incoming schema". These attributes will be greyed out. 
    -  Click "Add Attributes" twice. For the first attribute name enter "result_class" with "Number" as the type, and "prediction" as model field. For the second attribute name enter "predictions" with "Text as the type, and "probability" as the model field. The attributes list should now appear similar to the image below.
    - For more information, please click [here](https://www.ibm.com/support/producthub/icpdata/docs/content/SSQNUZ_current/wsj/streaming-pipelines/wml_operator.html)
    - <img src="https://github.com/IBMStreams/sample.edge-mnist-flows/blob/main/screenshots/output_schema.png" width="500">
  - Set up Kafka
    - As said earlier, the edge nodes send metrics back for further analysis. For this sample, we use 1 kafka topic to send both metrics on low scoring images, and aggregate metrics from all the data being scored.
    - If you already have your own kafka implementation, feel free to use that, if not, you can sign up for a free IBM cloud account by clicking [here](https://www.ibm.com/cloud/free)
      - After you sign up, you can create an Event Streams instance by clicking the blue "Create Resource" button and searching for "Event Streams", and following the given instructions.
      - Be sure to create a topic and service credentials to be used in the Streams flow
    - Back in the Streams Flows, the kafka node titled "Low confidence metrics - Kafka" is the one used to send low scoring images, while the node titled "Aggregate metrics - Kafka" sends aggregate metrics from all the data being scored
    - To add a connection click on either kafka node, and in the right panel select "Add connection" and populate it with the appropriate details. (e.g. if you're using Event Streams, populate the brokers, username, and password). When done, click "Create"
    - In the Settings panel, select the topic you have created. Now select the other node and fill in the same connection and topic settings.
    - For more information, please click [here](https://www.ibm.com/support/producthub/icpdata/docs/content/SSQNUZ_current/wsj/streaming-pipelines/Kafka.html)
  - Ensure necessary packages are installed and parameters are populated
    - While still in the Streams Flows edit page, select the settings button in the top left of the toolbar
    - Make sure that both panels have parameters and python packages set as in the image below
    - <img src="https://github.com/IBMStreams/sample.edge-mnist-flows/blob/main/screenshots/settings.png" width="250"> | <img src="https://github.com/IBMStreams/sample.edge-mnist-flows/blob/main/screenshots/packages.png" width="250"> 
  - As a final check to make sure everything is working, you can run the Streams Flows app
    - To do this first ensure that there are no red dots on any of the nodes. This indicates an error that requires fixing.
    - If there are no red dots, select the "save and run" button in the top left of the toolbar
    - Let the application run for a few minutes. Be sure to check the bell icon in the top right corner to see if there are any errors with the application.
    - If there are any errors, view the error log by selecting the bell icon, and then edit the app to fix the error
    - For more information, please click [here](https://www.ibm.com/support/producthub/icpdata/docs/content/SSQNUZ_current/wsj/streaming-pipelines/running-monitoring-streaming-pipeline.html)
   
    
#### 3. Build and deploy to the edge 
  - Build the edge app
    - If there are no errors, click the stop icon to quit the app
    - Now click the "Edit the Streams Flows" button, and in the top left corner click the "Build as Edge Analytics Application" button
    - For further instruction, see [here](https://www.ibm.com/support/knowledgecenter/SSQNUZ_3.0.1/svc-edge/developing-build.html)
  - Package the edge app
    - Once the edge app is built, we need to package it and prepare it for edge deployment. The link [here](https://www.ibm.com/support/knowledgecenter/SSQNUZ_3.0.1/svc-edge/usage-register-app.html) has more information on this
  - Deploying to the edge
    - Finally we can deploy the app to the edge, for more information, please click [here](https://www.ibm.com/support/knowledgecenter/SSQNUZ_3.0.1/svc-edge/usage-deploy.html)


#### 4. Notebook app
- After deploying the Streams Flow app to the edge, we can now create the IBM Streams app in Cloud Pak for Data that receives and process the scored images from the edge
- To do this, go back into your CPD project, and launch Jupyterlab IDE
- In the same directory as before `project_git_repo/<github-repo-name>/assets/jupyterlab/`, open `build-metro-app.ipynb`
  - Fill in the Streams instance name, Kafka topic name, and follow the instructions in the cells to input Kafka credentials. Run the rest of the cells to build and start the application.
- Once the application is up and running, open `render-metro-view.ipynb`, fill in the Streams instance name and run the cells to  preview the data.
