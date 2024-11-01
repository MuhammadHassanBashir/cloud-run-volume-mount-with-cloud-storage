# cloud-run-volume-mount-with-cloud-storage

link: https://cloud.google.com/run/docs/configuring/services/cloud-storage-volume-mounts#gcloud

To build a basic Python application that writes to a file in a Cloud Storage bucket mounted as a volume in Cloud Run, you can follow the steps below. Here’s how to set up the Python application, create a Docker container, and deploy it to Cloud Run.

1. Python Application Structure
Create a project directory with the following structure:


    cloud_run_volume_app/
    ├── app.py         # Main application code
    ├── Dockerfile     # Dockerfile to containerize the app
    ├── requirements.txt  # Dependency file

2. Code Implementation

app.py
This is the main application code that will write to the Cloud Storage volume.

        
        import os
        from flask import Flask
        
        app = Flask(__name__)
        
        # Define the mount path (should match the mount path used in `gcloud` command)
        mount_path = "/mnt/my-volume"
        
        # Ensure the directory exists
        if not os.path.exists(mount_path):
            os.makedirs(mount_path)
        
        # Define the file path
        file_path = os.path.join(mount_path, "example-log.txt")
        
        @app.route("/")
        def write_file():
            try:
                with open(file_path, "w") as f:
                    f.write("Hello, Cloud Storage! This is a test file stored via Cloud Run volume.\n")
                return f"File successfully written to {file_path}\n", 200
            except Exception as e:
                return f"Failed to write to the file: {str(e)}\n", 500
        
        if __name__ == "__main__":
            app.run(host="0.0.0.0", port=8080)

This application uses Flask to create an HTTP server with a single endpoint (/) that writes to example-log.txt in the mounted Cloud Storage volume.

requirements.txt

        Include Flask as a dependency:
        
        Flask

3. Dockerize the Application
        Create a Dockerfile to containerize the application.
        
        Dockerfile
        Dockerfile
        Copy code
        # Use the official Python image.
        FROM python:3.9-slim
        
        # Set environment variables for Flask
        ENV PYTHONUNBUFFERED True
        ENV PORT 8080
        
        # Install dependencies
        COPY requirements.txt .
        RUN pip install -r requirements.txt
        
        # Copy local code to the container image
        COPY . /app
        WORKDIR /app
        
        # Run the web server
        CMD ["python", "app.py"]

4. Build and Deploy Steps

    Step 1: Build the Docker Image
    In the terminal, navigate to the cloud_run_volume_app directory and build the Docker image:
    
    docker build -t gcr.io/world-learning-400909/cloud-run-volume-app .
    Replace YOUR_PROJECT_ID with your actual Google Cloud Project ID.
    
    Step 2: Push the Docker Image to Artifact Registry
    First, ensure Artifact Registry is enabled in your Google Cloud project and that you’re authenticated to push images.

    gcloud auth configure-docker
    docker push gcr.io/world-learning-400909/cloud-run-volume-app

Step 3: Deploy to Cloud Run with Volume Mount
    
    Use the gcloud command to deploy the container to Cloud Run, specifying the volume mount:
    
    gcloud run deploy cloud-run-volume-app \
    --image gcr.io/world-learning-400909/cloud-run-volume-app \
    --region us-central1 \
    --allow-unauthenticated \
    --add-volume name=volume1,type=cloud-storage,bucket=testbucket12131 \
    --add-volume-mount volume=volume1,mount-path=/mnt/my-volume

        
    YOUR_PROJECT_ID with your Google Cloud Project ID.
    YOUR_BUCKET_NAME with the name of your Cloud Storage bucket.

5. Test the Application
    
    Once deployed, go to the Cloud Run service URL provided by Google Cloud and navigate to the root endpoint (e.g., https://your-cloud-run-service-url/). If successful, the application will write "Hello, Cloud Storage!" to the example-log.txt file in the mounted Cloud Storage volume.

6. Verify the File in Cloud Storage

    To check if the file was created, use the following gsutil command:

    gsutil ls gs://YOUR_BUCKET_NAME/example-log.txt
   
    If the file appears in your bucket, the Cloud Run volume mount setup is working correctly.
