# Project Phase- GADS GCP Cloud track

Here is a repo with details of the labs I took during the project phase of the GADS GCP Cloud track. The screenshots
folder contains screenshots of the confirmation email sent from Qwiklabs about the completion of a lab. I used the  [gcloud reference](https://cloud.google.com/sdk/gcloud/reference) to find the description of the gcloud commands.


## Lab 1: Creating Virtual Machines

### Objectives
  - Create several standard VMs
  - Create advanced VMs

### Tasks
To build what I built in the lab using the command line:  
 1. Creating a utility VM  
 ```
 export ZONE='us-central1-c'  
 export INSTANCE_NAME='my-new-instance'  
 export INSTANCE_TYPE='n1-standard-1'  
 gcloud compute instances create $INSTANCE_NAME --zone=$ZONE --machine-type=$INSTANCE_TYPE --network-interface=no-address
```

In the gcloud command, the 'no-address' value in the --network-interface option means that an external IP 
will not be created for this VM.

 2. Creating a Windows VM

 To find the name of the image that I needed I used the command `gcloud compute images list` which returns  
a full list of public images with their image names, versions numbers, and image sizes. 

 ```
 export ZONE='europe-west2-a'  
 export INSTANCE_NAME='windows-instance'  
 export INSTANCE_TYPE='n1-standard-2'  
 export IMAGE_FAMILY='windows-server-2016-dc-core-v20200813'  
 export IMAGE_PROJECT='windows-core'
 gcloud compute instances create $INSTANCE_NAME --zone=$ZONE --machine-type=$INSTANCE_TYPE --image-family=$IMAGE_FAMILY
  --image-project=$IMAGE_PROJECT
```

 3. Create a custom virtual machine  
 ```
 export ZONE='us-west1-b'    
 export INSTANCE_NAME='custom-instance'   
 export INSTANCE_TYPE='n1-standard-2'    
 gcloud compute instances create $INSTANCE_NAME --zone=$ZONE --custom-cpu=6 --custom-memory=32
```

## Lab 2: App Dev: Setting up a Development Environment v1.1 

### Objectives
 - Provision a Google Compute Engine instance.
 - Connect to the instance using SSH.
 - Install software on the instance.
 - Verify the software installation.


### Tasks 
  1. Creating a Compute Engine Virtual Machine Instance
  We are going to create a VM instance will full access scopes and allow http traffic
```
 export ZONE='us-central1-a'    
 export INSTANCE_NAME='dev-instance'   
 gcloud compute instances create $INSTANCE_NAME --zone=$ZONE --scopes=cloud-platform --subnet "default" --tags http-server    
```

  We take note of the VM's external IP address.

  Then we create a firewall rule to allow http traffic on that VM
```
gcloud compute --project=XXXX firewall-rules create default-allow-http --direction=INGRESS --priority=1000 --network=default --action=ALLOW 
--rules=tcp:80 --source-ranges=0.0.0.0/0 --target-tags=http-server
```

  2. Install software on the VM instance
  We first ssh into the VM we just created. After running the command, we can accept the prompts and leave `passphrase` blank
```
  gcloud compute ssh $INSTANCE_NAME --zone=$ZONE 
```
  Next we update packages and install git and node
```
sudo apt-get update 
```
```
sudo apt-get install git 
```
```
curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash - 
```
```
sudo apt install nodejs 
```

 3. Configure the VM to Run Application Software
 We check the version of node installede
```
  node -v
```

  Then we clone the class repository
```
 git clone https://github.com/GoogleCloudPlatform/training-data-analyst 
```

  We then, change to the repository's root directory
```
  cd ~/training-data-analyst/courses/developingapps/nodejs/devenv/
```

  Next, we run the node server
```
  sudo node server/app.js
```

  We can test that the application is running using curl and the external IP address of the VM instance
```
  curl <vm-ip-address>
```

## Lab 3: Google Cloud Fundamentals: Getting Started with Cloud Storage and Cloud SQL 

### Objectives
 - Create a Cloud Storage bucket and place an image into it.
 - Create a Cloud SQL instance and configure it.
 - Connect to the Cloud SQL instance from a web server.
 - Use the image in the Cloud Storage bucket on a web page.

### Tasks
  1. Sign in to Google Cloud Console with the credentials provided
   We might be asked to set the project in Cloud Shell. Use the project ID given
```
gcloud config set project PROJECT_ID
```

  2. Deploy a web server VM instance 
  First we would need to create a startup script for the VM. We will use nano editor to create the file. 

```
nano startup.sh
```

  We then paste the following into the editor then press `Ctrl + O` to save and `Ctrl + X` to exit the editor:

```
apt-get update
apt-get install apache2 php php-mysql -y
service apache2 restart
```

  Next we run the following command to create the VM instance.

```
 export VM_INSTANCE_NAME='bloghost'  
 export IMAGE_NAME='debian-9-stretch-v20200902'  
 export IMAGE_PROJECT='debian-cloud'
 export ZONE='us-central1-a'
 gcloud compute instances create $VM_INSTANCE_NAME --zone=$ZONE --image-project=$IMAGE_PROJECT --image=$IMAGE_NAME --subnet "default" --tags http-server --metadata-from-file startup-script=startup.sh
```

  We take note of the internal and external IP address of the VM instance from the output printed out.

  Then we create a firewall rule to allow http traffic on that VM

```
gcloud compute --project=XXXX firewall-rules create default-allow-http --direction=INGRESS --priority=1000 --network=default --action=ALLOW 
--rules=tcp:80 --source-ranges=0.0.0.0/0 --target-tags=http-server
```

  Lastly, we run a startup script.

```
    sudo apt-get update 
    sudo apt-get install apache2 php php-mysql -y 
    service apache2 restart
```

  3. Create a Cloud Storage bucket using the gsutil command line
  We create a storage bucket and give it a name which is the project ID 

```
    export LOCATION=US
    gsutil mb -l $LOCATION gs://$DEVSHELL_PROJECT_ID
```

  Retrieve a banner image from a publicly accessible Cloud Storage location:

```
gsutil cp gs://cloud-training/gcpfci/my-excellent-blog.png my-excellent-blog.png
```

  Copy the banner image to your newly created Cloud Storage bucket:

```
gsutil cp my-excellent-blog.png gs://$DEVSHELL_PROJECT_ID/my-excellent-blog.png
```

  Modify the Access Control List of the object you just created so that it is readable by everyone:

```
gsutil acl ch -u allUsers:R gs://$DEVSHELL_PROJECT_ID/my-excellent-blog.png
```


  4. Create the Cloud SQL instance
  We use the external IP address of the VM instance we created earlier as a network for the SQL instance we create. We append `/32` to the IP address

```
    export DB_INSTANCE_NAME='blog-db'
    export VM_INSTANCE_IP='35.202.51.110/32'
    export ROOT_PASSWORD='StrongPasswordHere'
    gcloud sql instances create $DB_INSTANCE_NAME --zone=$ZONE --database-version=MYSQL_5_7 --root-password=$ROOT_PASSWORD --authorized-networks=$VM_INSTANCE 
```

  After running this command, details of the instance will be returned. We will use the `Primary Address` that we are given.

  5. Configure an application in a Compute Engine instance to use Cloud SQL
  First we ssh into our VM

```
    gcloud compute ssh $VM_INSTANCE_NAME
```

  Once in the instance we change the working directory to the document root of the web server
```
    cd /var/www/html
```
  Then we create a index.php file
```
  sudo nano index.php
```

   next we paste in code to connect with the MySQL database
```
<html>
<head><title>Welcome to my excellent blog</title></head>
<body>
<h1>Welcome to my excellent blog</h1>
<?php
 $dbserver = "CLOUDSQLIP";
$dbuser = "blogdbuser";
$dbpassword = "DBPASSWORD";
// In a production blog, we would not store the MySQL
// password in the document root. Instead, we would store it in a
// configuration file elsewhere on the web server VM instance.

$conn = new mysqli($dbserver, $dbuser, $dbpassword);

if (mysqli_connect_error()) {
        echo ("Database connection failed: " . mysqli_connect_error());
} else {
        echo ("Database connection succeeded.");
}
?>
</body></html>

```
  We then restart the web server
```
  sudo service apache2 restart
```

  Using the browser, we navigate to where our index.php file can be seen using the external IP address of the VM. 
  To get the external IP, we run a command to list the external IPs for our VM

```
  gcloud compute addresses list
```
  Once we have the external IP, we can view the index.php file in the browser
``` 
  <external-ip-address>/index.php
```

  In the broswer, we should see a connection error. Open nano editor and replace `CLOUDSQLIP` with the `Primary Address` of the
  SQL instance and `DBPASSWORD` with the password we noted earlier. 
  
  Restart the web server
```
  sudo service apache2 restart
```
  
  We should see that we have successfully connected to the database

  6. Configure an application in a Compute Engine instance to use a Cloud Storage object

  We are first going to get our storage bucket name. We run a command to list the buckets in our project
```
  gsutil ls
```

  The we list the objects in our storage bucket
```
 gsutil ls -r gs://BUCKET_NAME/**
```
  
  We should see that we have a file named, "my-excellent-blog.png". We are going to make it publicly readable by running this command.
```
  gsutil acl ch -u AllUsers:R gs://[BUCKET_NAME]/my-excellent-blog.png
```
  
  After executing that command, we should get the publicly accessible link to the file. We then return to the index.php
  file we created in the previous steps and add an `img` tag and give the src prop the link to "my-excellent-blog.png"
  in our bucket. 
  
  Last we restart the server
```
  sudo service apache2 restart
```

  When we navigate to the index.php file in our browser, we should see the banner image. 
  
  That is all for these labs. If you have a query, feel free to create an issue.

