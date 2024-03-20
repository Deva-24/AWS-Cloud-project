*Detailed Execution workflow of AWS SQS PIPELINE with screenshots are in Assignment PDF in the same repository.*

The above projects contain code to recognize images and text using AWS tools. They also include code for communicating via SQS, a messaging service in AWS. This communication logic is intertwined with the image and text recognition code because they need to work together on the EC2 instances for everything to function simultaneously.

**STEPS:**
Log into AWS Student Account.
Start Lab in the vocareum and click on the green signal of AWS button to start the console. 
This will automatically head over to the AWS Management console 

**Process of Creating EC2 Instances:** 
**Navigate to EC2 Dashboard:** From the services menu, select EC2 to navigate to the EC2 Dashboard.
**Launch Instance:** Click on the "Launch Instance" button to start the process of creating a new EC2 instance.
**Choose Linux AMI:** In the "Choose an Amazon Machine Image (AMI)" step, select "Amazon Linux 2 AMI (HVM) - Kernel 5.10, SSD Volume Type". This is a stable and widely used AMI for general-purpose computing.
**Choose default Instance Type:** Select the instance type to be "t2.micro". This is a low-cost, general-purpose instance type suitable for small applications and testing purposes.
**Create Key Pair:** Create a new key-pair. This key pair will be used to securely connect to your instance via SSH from a local machine.
**Configure Security Group:** Create a new security group with the following settings:
    Allow SSH traffic (port 22) from your IP address (select "My IP").
    Allow HTTP traffic (port 80) from the internet.
    Allow HTTPS traffic (port 443) from the internet.
**Advanced Options -> IAM Role:** Select the LabInstanceRole in the dropdown of AWS Student account.
**Launch Instance:** Click on the "Launch" button to launch the instance after ensuring all the correct configurations are made.

This should be done twice to create two seperate instances **abi_image_rekognition** and **abi_text_rekognition** 

**Accessing Instance:** Once the instance is launched, you can access it using SSH. Use the public IP address or public DNS provided by AWS to connect to your instance.
    Use CyberDuck to connect to SSH from mac and Putty or WinSCP from windows.
    Select SFTP which is a file transfer protocol to upload to EC2 via cyberduck. 
    Enter Public DNS and then attach the .pem file of the respective EC2 wanted to connect. 
    Once connected , drag and drop the JAR files of the projects separately to respective EC2s.

**Connecting SSH:** The below steps to be done to both EC2s created

Headover to the EC2 and click on connect, and select SSH Client Tab. 
Open the putty and enter the IP address of the instance and configure the .ppk file assigned for that instance.
enter the login user as ec2-user and click on connect.

The addresses change for different EC2s and the same action needs to be done to connect two EC2s that were created.

After connecting to each EC2, 

Type in the command ->  **ls** to see whether the uploaded jar file is present or not 

To Install Java in linux, execute the below 2 commands 
    sudo amazon-linux-extras install java-openjdk11 -y
    sudo yum install java-1.8.0-openjdk -y

To check if java is successfully installed -> java -version

In the Vocareum, our AWS credentials are present in the AWS Details Tab. 
It is required to create aws credentials file and can be done using the below commands 
    To create directory         -> mkdir home/ec2-user/.aws
    To create a file            -> touch home/ec2-user/.aws/credentials 
    To write in the credentials -> vi ~/.aws/credentials -> Paste the credentials and press ESC key and enter **:wq** to save the file


**Execution of JAR Files:**

**Commands to Execute JAR files:**

*First* : Run ImageRekognition JAR in abi_image_rekognition instance 
	-> java -jar image-rekognition-1.0-SNAPSHOT.jar 
*Second* : Run TextRekognition JAR in abi_text_rekognition instance 
	-> java -jar text-rekognition-1.0-SNAPSHOT.jar

**abi_image_rekognition**:
This component is responsible for processing images using Amazon Rekognition.
It starts by fetching car images from a public S3 bucket (https://njit-cs-643.s3.us-east-1.amazonaws.com).
Once an image is fetched, it performs object recognition on the image using Amazon Rekognition.
After processing, it prints the list of objects detected in the image.
Finally, it pushes the processed image and its results to an SQS Queue.

**abi_text_rekognition:**
This component is responsible for processing text from car images.
It listens to the SQS Queue where the processed images are pushed by abi_image_rekognition.
When an image is received from the queue, it fetches the respective image.
Then, it performs text recognition on the image using Amazon Rekognition.
After processing, it recognizes the text present in the image and prints the output.

**SQS Queue:** The SQS Queue is used as a mechanism for communication between abi_image_rekognition and abi_text_rekognition, allowing the latter to know when new images are ready for text recognition.

**Workflow in Instances:** 

Started by executing abi_image_rekognition to process the car images and push the results to the SQS Queue.

Once the abi_image_rekognition is executed, it creates a Queue from code.
In my case, code will initiate queue av739.fifo (only if not exists, so it doesn't create multiple times when you run again) and this created a queue automatically in AWS and uses it. 
Navigate to services in AWS console and search for SQS, and this shows up that QUEUE is created and also shows the statistics in graph metrics. 

Next, executed  abi_text_rekognition to listen to the SQS Queue and process the images for text recognition.
abi_text_rekognition fetches the processed images and recognizes the text present in the images.

Finally, it prints the output of the text recognition process.





