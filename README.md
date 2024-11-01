# cloud-run-volume-mount-with-cloud-storage

## Cloud Run Volume Mount with Cloud Storage - Complete Guide

This guide covers:

Requirements and Setup
Creating a Cloud Storage Volume Mount
Writing and Reading Files in the Mounted Volume
Verifying File Creation in Cloud Storage

1. Requirements and Setup

    To mount a Cloud Storage bucket as a volume in Cloud Run, ensure the following:
    
    Cloud Run service with the Second Generation execution environment (automatically used if not explicitly configured).
    
    Permissions: The Cloud Run service account must have access to the Cloud Storage bucket. The Storage Admin role can be applied to the service account, allowing read/write access to the bucket.
    Google Cloud CLI (gcloud): This is used to configure and deploy services.

2. Creating a Cloud Storage Volume Mount in Cloud Run

    You can use the Cloud Console or gcloud commands to mount a Cloud Storage bucket as a volume.
    
    Using the gcloud CLI:
    
    gcloud run services update SERVICE_NAME \
        --add-volume name=VOLUME_NAME,type=cloud-storage,bucket=BUCKET_NAME \
        --add-volume-mount volume=VOLUME_NAME,mount-path=/mnt/my-volume
    Replace SERVICE_NAME with your Cloud Run service name.
    Replace VOLUME_NAME with a name for the volume.
    Replace BUCKET_NAME with the name of your Cloud Storage bucket.
    Replace /mnt/my-volume with the path where you want to mount the volume in your container.
    Note: If you have multiple containers, add volume mounts for each container as needed.
    
    To make the volume read-only (if you don't intend to write to it):
    
    gcloud run services update SERVICE_NAME \
        --add-volume name=VOLUME_NAME,type=cloud-storage,bucket=BUCKET_NAME,readonly=true \
        --add-volume-mount volume=VOLUME_NAME,mount-path=/mnt/my-volume

3. Writing and Reading Files in the Mounted Volume

    Now that you have mounted the Cloud Storage volume, you can write and read files from the specified path in your code.
    
    Example Python code to create and write data to a file in the mounted volume:
    
    python
    Copy code
    import os
    
    # Define the mount path, this should match the mount path used in gcloud command
    mount_path = "/mnt/my-volume"
    
    # Ensure the directory exists in case it needs to create it within the container
    if not os.path.exists(mount_path):
        os.makedirs(mount_path)
    
    # Path to the file you want to create
    file_path = os.path.join(mount_path, "example-log.txt")
    
    try:
        # Open the file in write mode and add some content
        with open(file_path, "w") as f:
            f.write("Hello, Cloud Storage! This is a test file stored via Cloud Run volume.\n")
        print(f"File successfully written to {file_path}")
    except Exception as e:
        print(f"Failed to write to the file: {str(e)}")
    To deploy this code to Cloud Run, include it in your service code and run:
    
    bash
    Copy code
    gcloud run deploy SERVICE_NAME --source .

4. Verifying File Creation in Cloud Storage

    After the service has successfully run, the created file should be visible in your Cloud Storage bucket. Use the following gsutil command to check:
    
    gsutil ls gs://BUCKET_NAME/example-log.txt
    Replace BUCKET_NAME with the actual bucket name you used in the setup. If the file example-log.txt is in the bucket, your setup is complete and working.
    
    Additional Tips
    Multiple Buckets: You can mount multiple Cloud Storage buckets by adding more volume mounts, each with unique paths.
    Container Permissions: Ensure the Cloud Run service account has Storage Object Creator or higher permissions for writing and Storage Object Viewer for reading, depending on access needs.
    Testing Changes: If updating volume mounts or paths, always redeploy the service and verify Cloud Run has the correct configurations.
    This completes the setup for persisting data between your Cloud Run container and Cloud Storage using volume mounts.
