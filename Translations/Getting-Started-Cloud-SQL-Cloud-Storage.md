# LAB Google Cloud Fundamentals: Getting Started with Cloud Storage and Cloud SQL

## Objectives

In this lab, you learn how to perform the following tasks:

    - Create a Cloud Storage bucket and place an image into it.

    - Create a Cloud SQL instance and configure it.

    - Connect to the Cloud SQL instance from a web server.

    - Use the image in the Cloud Storage bucket on a web page.

## Steps

1. Create a Cloud Storage bucket and place an image into it

    - In case the project id is not set. The project id can be obtained from the qwiklabs lab launch page.

        gcloud config set project {PROJECT-ID}

    - Create and save a startup script file in notepad with the following content

        apt-get update
        apt-get install apache2 php php-mysql -y
        service apache2 restart

    - In Cloud Shell, Click the three dots near the avatar icon. Then select Upload file to upload the above startup script file

        Result: Upload successful

    - Deploy a web server VM instance using the below command, passing the path of the startup script. In case it is in the present directory, do not change the below command.

        gcloud compute instances create bloghost --zone=us-central1-a --tags=http-server --metadata-from-file=startup-script=./startup-script.txt

    - Allow http connections to the server

        gcloud compute firewall-rules create default-allow-http --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:80 --source-ranges=0.0.0.0/0 --target-tags=http-server

    - Export the location of the lab

        export LOCATION=US

    - Create a bucket in the above location

        gsutil mb -l $LOCATION gs://$DEVSHELL_PROJECT_ID

    - Retrieve a banner image from a publicly accessible Cloud Storage location:

        gsutil cp gs://cloud-training/gcpfci/my-excellent-blog.png my-excellent-blog.png

    - Copy the banner image to your newly created Cloud Storage bucket:

        gsutil cp my-excellent-blog.png gs://$DEVSHELL_PROJECT_ID/my-excellent-blog.png

    - Modify the Access Control List of the object you just created so that it is readable by everyone:

        gsutil acl ch -u allUsers:R gs://$DEVSHELL_PROJECT_ID/my-excellent-blog.png

2. Create a Cloud SQL instance and configure it

    - Create the SQL instance in the same zone as the vm for better performance

        gcloud sql instances create blog-db --zone=us-central1-a --root-password=blog-db

    - After it has been created, copy the public address to a text editor for later use. It is labelled as primary address.

    - Create a user to be used with the database

        gcloud sql users create blogdbuser --instance=blog-db --password=blogdbuser

    - Add the public IP address of the vm to the list of authorized networks

        gcloud sql instances patch blog-db --authorized-networks=104.198.128.106/32

3. Connect to the Cloud SQL instance from a web server

    - Connect to the vm using SSH

        gcloud compute ssh bloghost --zone=us-central1-a

        Enter Y to confirm. Then enter the passphrase twice to connect.

    - Navigate to the document root directory

        cd /var/www/html

    - Create an index.php file and open it using nano

        sudo nano index.php

    - Paste the following into the file

        Refer to the HTML content section near the bottom of this page for the code

    - Press Ctrl+O, and then press Enter to save your edited file

    - Press Ctrl+X to exit the nano text editor

    - Restart the web server:

        sudo service apache2 restart

    - Open a new web browser tab and paste into the address bar your bloghost VM instance's external IP address followed by /index.php.

    - When you load the page, you will see that its content includes an error message beginning with the words:

        Database connection failed: ...

    - Edit the file to add the Cloud SQL instance IP

        sudo nano index.php

    - In the nano text editor, replace CLOUDSQLIP with the Cloud SQL instance Public IP address that you noted above. Leave the quotation marks around the value in place.

    - In the nano text editor, replace DBPASSWORD with the Cloud SQL database password that you defined above. Leave the quotation marks around the value in place.

    - Press Ctrl+O, and then press Enter to save your edited file.

    - Press Ctrl+X to exit the nano text editor.

    - Restart the web server:

        sudo service apache2 restart

    - Return to the web browser tab in which you opened your bloghost VM instance's external IP address. When you load the page, the following message appears:

        Database connection succeeded.

4. Use the image in the Cloud Storage bucket on a web page

    - Use the nano text editor to edit index.php:
        sudo nano index.php

    - Cloud storage buckets use the following URL format after public access has been provided without < > brackets.

        <https://storage.googleapis.com/>my-bucket-name/file-path

    - For instance, if the public access was provided successfully, the following url will be valid. Replace $DEVSHELL_PROJECT_ID with your project id since this variable is not recognizable within the vm. Remove < > brackets.

        <https://storage.googleapis.com/$DEVSHELL_PROJECT_ID/my-excellent-blog.png>

    - Use the arrow keys to move the cursor to the line that contains the h1 element. Press Enter to open up a new, blank screen line, and then paste the above modified URL into the line.

    - Paste this HTML markup immediately before the URL:

        <img src='

    - Place a closing single quotation mark and a closing angle bracket at the end of the URL:

        '>

    - Press Ctrl+O, and then press Enter to save your edited file.

    - Press Ctrl+X to exit the nano text editor.

    - Restart the web server:

        sudo service apache2 restart

    - Return to the web browser tab in which you opened your bloghost VM instance's external IP address. When you load the page, its content now includes a banner image.

## HTML content section

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

            </body>
        </html>
