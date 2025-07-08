Below is a comprehensive, step‐by‐step guide for setting up and executing your experiments in AWS. Each experiment is broken down into its tasks with clear instructions, AWS console actions, and command‐line examples where applicable. Be sure you have the necessary permissions (and key pairs created) in your AWS account before proceeding.

---

## Experiment 1: Working with EC2 Instances

### Task 1: Launch an EC2 Instance Using Ubuntu OS, Connect via SSH, and Use Direct Connect

1. **Launch the Instance:**
   - Log in to the AWS Management Console and navigate to **EC2**.
   - Click **Launch Instance**.
   - **Choose an AMI:** Select an official Ubuntu Server (e.g., *Ubuntu Server 20.04 LTS*).
   - **Instance Type:** Choose an instance type (e.g., t2.micro).
   - **Configure Instance Details:** Adjust settings if needed (leave the defaults for a basic experiment).  
   - **Add Storage:** Accept default volume size or increase as needed.
   - **Add Tags:** Optionally add a tag (e.g., Key: `Name`, Value: `Ubuntu-SSH`).
   - **Configure Security Group:**  
     - Create or select a security group that allows inbound SSH (port 22) from your IP.
   - **Key Pair:** Create a new key pair or choose an existing one, download the PEM file, and secure it (set permissions with `chmod 400 your-key.pem`).

2. **Connect to the Instance via SSH:**
   - In the EC2 console, select your instance and click **Connect**.
   - Follow the instructions or use your terminal:
     ```bash
     ssh -i "your-key.pem" ubuntu@<Public-IP-of-Instance>
     ```
   - **Note on Direct Connect:** If you refer to using [AWS Direct Connect](https://aws.amazon.com/directconnect/), ensure that your on-premises connectivity is established as per AWS guidelines. Otherwise, “direct connect” might simply indicate connecting directly via SSH.

### Task 2: Install Nginx on the Ubuntu Instance and Verify via Browser

1. **Update and Install Nginx:**
   - Once connected via SSH:
     ```bash
     sudo apt update
     sudo apt install nginx -y
     ```
2. **Allow HTTP Traffic:**
   - Modify your instance’s security group (via the AWS console) to allow inbound HTTP (port 80) traffic from your IP or 0.0.0.0/0 for public access.
3. **Verify Nginx:**
   - Ensure Nginx is running:
     ```bash
     sudo systemctl status nginx
     ```
   - Open a web browser and navigate to `http://<Public-IP-of-Instance>` – you should see the Nginx welcome page.

### Task 3: Launch a New Instance Using Amazon Linux and Connect Using PuTTY

1. **Launch an Amazon Linux Instance:**
   - In the EC2 console, click **Launch Instance**.
   - **Choose an AMI:** Select Amazon Linux 2.
   - Follow the steps as in Task 1 (choose instance type, storage, tags, etc.).
   - **Security Group:** Ensure port 22 is allowed.
   - **Key Pair:** If you plan to use PuTTY, convert your PEM key to a PPK file using [PuTTYgen](https://www.puttygen.com/).
2. **Connect via PuTTY:**
   - Open PuTTY and enter the **Public IP** of your instance in the “Host Name” field.
   - Under **Connection > SSH > Auth**, browse and select your converted PPK file.
   - Click **Open** to establish the connection.
   - Log in using the default username for Amazon Linux (typically `ec2-user`).

### Task 4: Launch an EC2 Instance Using Windows OS and Connect via RDP

1. **Launch a Windows Instance:**
   - In the EC2 console, click **Launch Instance**.
   - **Choose an AMI:** Select a Windows Server (e.g., *Microsoft Windows Server 2019 Base*).
   - Follow the remaining launch steps (instance type, storage, tags).
   - **Security Group:** Ensure you allow RDP (port 3389) from your IP.
   - **Key Pair:** Use an existing key pair or create a new one.
2. **Connect Using RDP:**
   - Once the instance is running, select it and click **Connect** > **Get Windows Password**.
   - Wait a few minutes for the password to be available.
   - Decrypt the administrator password using your PEM key.
   - Download the remote desktop file and use the Windows Remote Desktop Client (or equivalent) to connect using the Administrator credentials.

---

## Experiment 2: Deploy a Front-End Application Using Docker

### Task 1: Install Docker on an EC2 Instance

1. **Launch an EC2 Instance:**
   - You may use Ubuntu or Amazon Linux.
   - Ensure that the instance’s security group allows relevant web traffic (HTTP/HTTPS) and SSH.
2. **Install Docker (for Ubuntu example):**
   ```bash
   sudo apt update
   sudo apt install -y docker.io
   sudo systemctl enable docker
   sudo systemctl start docker
   ```
   *For Amazon Linux 2:*
   ```bash
   sudo amazon-linux-extras install docker -y
   sudo service docker start
   sudo usermod -a -G docker ec2-user
   ```
   - Log out and log back in for group changes to take effect.

### Task 2: Clone the Front-End Application, Create a Dockerfile, and Build the Image

1. **Clone the Repository:**
   - Connect via SSH to your EC2 instance.
   - Clone the repository using Git. For example:
     ```bash
     git clone https://tinyurl.com/demokmit3
     cd <repository-directory>
     ```
2. **Create a Dockerfile:**
   - Inside the repository folder, create a file named `Dockerfile` (if not already provided) with contents similar to:
     ```Dockerfile
     # Use an official Node.js (or other appropriate) image
     FROM node:14-alpine
     WORKDIR /app
     COPY package*.json ./
     RUN npm install
     COPY . .
     EXPOSE 80
     CMD ["npm", "start"]
     ```
   - Adjust commands based on the application’s requirements.
3. **Build the Docker Image:**
   ```bash
   docker build -t frontend-app .
   ```

### Task 3: Create a Docker Container Running the Front-End Application

1. **Run the Container:**
   ```bash
   docker run -d -p 80:80 frontend-app
   ```
2. **Test the Application:**
   - Open a web browser and navigate to `http://<Public-IP-of-Instance>` to verify that the front-end application is running.

### Task 4: Configure Security Groups to Allow Web Traffic

1. **Edit the Security Group:**
   - In the AWS EC2 console, find the Security Group attached to your instance.
   - Add an inbound rule:
     - **Type:** HTTP  
     - **Port Range:** 80  
     - **Source:** Anywhere (0.0.0.0/0) or restrict to your IP as needed.
   - Save the changes.

---

## Experiment 3: Create and Configure Storage Services Using Amazon EBS

### Task 1: Launch an EC2 Instance, Check the Volume, and Modify the Volume Size

1. **Launch Instance:**
   - Launch an EC2 instance (Ubuntu, Amazon Linux, etc.) from the console.
2. **Check Volume:**
   - In the console, under **Elastic Block Store > Volumes**, verify the attached root volume size.
3. **Modify the Volume:**
   - Select the volume, click **Actions > Modify Volume**.
   - Change the size as required and click **Modify**.
   - On the instance, use OS-level tools (e.g., `lsblk` in Linux) to see the block device, then expand the file system:
     - For ext4:
       ```bash
       sudo resize2fs /dev/xvda1
       ```

### Task 2: Create a New Volume, Attach it, Mount, and Create a File

1. **Create a New Volume:**
   - In the EC2 console, go to **Volumes** and click **Create Volume**.
   - Specify the size, type, and ensure it is in the same Availability Zone as your instance.
2. **Attach the Volume:**
   - Select the newly created volume, click **Actions > Attach Volume**, and choose your instance.
3. **Mount the New Volume on the EC2 Instance:**
   - SSH into the instance.
   - Identify the volume (e.g., `/dev/xvdf`), then:
     ```bash
     sudo mkfs -t ext4 /dev/xvdf       # Format if not preformatted
     sudo mkdir /mnt/newvolume
     sudo mount /dev/xvdf /mnt/newvolume
     ```
4. **Create a File:**
   - Create a file with your roll number:
     ```bash
     echo "YourRollNo" | sudo tee /mnt/newvolume/YourRollNo.txt
     ```

### Task 3: Detach the Volume and Reattach to Another EC2 Instance

1. **Detach the Volume:**
   - In the console, select the volume and choose **Actions > Detach Volume**.
2. **Attach to New Instance:**
   - Launch (or choose) a different EC2 instance in the same Availability Zone.
   - Attach the volume to this new instance.
3. **Verify the File:**
   - SSH into the new instance.
   - Mount the volume as before:
     ```bash
     sudo mkdir /mnt/volume2
     sudo mount /dev/xvdf /mnt/volume2
     ls -l /mnt/volume2
     ```
   - Confirm that your roll number file is present.

### Task 4: Create a Snapshot, Copy to Another Region, and Attach to an Instance

1. **Create a Snapshot:**
   - In the console, select the volume and choose **Actions > Create Snapshot**.
   - Provide a name/description and create the snapshot.
2. **Copy the Snapshot to Another Region:**
   - Go to **Snapshots**, select your snapshot, and choose **Copy**.
   - Select the destination region and start the copy process.
3. **Create a New Volume from the Snapshot in the New Region:**
   - In the destination region, navigate to **Snapshots**, select the copied snapshot, and choose **Create Volume**.
4. **Attach the Volume:**
   - Launch an EC2 instance in the new region.
   - Attach the newly created volume and mount it following the same mounting steps to verify your data.

---

## Experiment 4: Amazon Elastic File System (EFS)

### Task 1: Create Two Security Groups (One for EC2 and One for EFS)

1. **Security Group for EC2:**
   - Create a security group allowing SSH (port 22) and any additional ports needed (e.g., HTTP).
2. **Security Group for EFS:**
   - Create a security group for EFS that allows NFS traffic (port 2049) from the EC2 security group.
   - In the EFS security group rules, set the source to the EC2 security group’s ID.

### Task 2: Launch an EC2 Instance and Mount the EFS

1. **Launch an EC2 Instance:**
   - Use the EC2 console to launch an instance using your preferred AMI.
   - Assign the EC2 security group to it.
2. **Create EFS File System:**
   - Navigate to the **EFS** console and create a new file system.
   - During creation, assign the EFS security group.
   - Ensure the EFS is in the same VPC as your EC2 instance.
3. **Mount the EFS:**
   - In your EC2 instance, install the NFS client:
     ```bash
     sudo apt update && sudo apt install -y nfs-common   # For Ubuntu
     ```
   - Create a mount point and mount the EFS using its DNS name:
     ```bash
     sudo mkdir /mnt/efs
     sudo mount -t nfs4 -o nfsvers=4.1 <EFS-DNS-Name>:/ /mnt/efs
     ```

### Task 3: Create a Directory and a File with Your Roll Number

- On the mounted file system, run:
  ```bash
  sudo mkdir /mnt/efs/rollno_dir
  echo "YourRollNo" | sudo tee /mnt/efs/rollno_dir/YourRollNo.txt
  ```

### Task 4: Launch a Second EC2 Instance and Mount the Same EFS

1. **Launch a Second EC2 Instance:**
   - Use the same VPC and assign the same EC2 security group.
2. **Mount the Existing EFS:**
   - SSH into the second instance and mount the same EFS using the steps from Task 2.
   - Verify the directory and file:
     ```bash
     ls /mnt/efs/rollno_dir
     cat /mnt/efs/rollno_dir/YourRollNo.txt
     ```

---

## Experiment 5: Amazon VPC

### Task 1: Create a VPC

1. **Create VPC:**
   - In the AWS VPC console, click **Create VPC**.
   - Specify a CIDR block (e.g., 10.0.0.0/16) and provide a name tag.
  
### Task 2: Create Two Subnets (Public and Private)

1. **Public Subnet:**
   - Create a subnet within the VPC with a CIDR block (e.g., 10.0.1.0/24).
2. **Private Subnet:**
   - Create a second subnet (e.g., 10.0.2.0/24).

### Task 3: Launch Two EC2 Instances (One in Public, One in Private)

1. **Launch Instance in Public Subnet:**
   - When launching, under **Network settings**, choose your custom VPC and assign the public subnet.
   - Ensure it has a public IP.
2. **Launch Instance in Private Subnet:**
   - Launch a second instance in your VPC and assign it to the private subnet (no public IP).

### Task 4: Create and Attach an Internet Gateway (IGW)

1. **Create IGW:**
   - In the VPC console, click **Internet Gateways > Create Internet Gateway**.
   - Attach the IGW to your VPC.

### Task 5: Create Route Tables and Associate with the IGW

1. **Public Route Table:**
   - Create (or modify) a route table for the public subnet and add a route to 0.0.0.0/0 targeting the IGW.
   - Associate the public subnet with this route table.
2. **Private Route Table:**
   - The private subnet route table may not have direct internet access unless you set up NAT (next experiment).

### Task 6: Access the Instance in the Private Subnet from the Public Subnet

1. **Setup a Bastion Host:**
   - Use the EC2 instance in the public subnet as a jump server.
2. **SSH Tunneling:**
   - From your local machine, SSH into the public instance.
   - From the public instance, SSH into the private instance using its private IP.
   - Alternatively, use Session Manager if available.

---

## Experiment 6: Amazon VPC NAT Gateway

### Task 1: Create a VPC

- Follow the steps from Experiment 5, Task 1.

### Task 2: Create Two Subnets (Public and Private)

- Follow the Experiment 5, Task 2 steps.

### Task 3: Launch Two EC2 Instances

- Launch one instance in the public subnet and one in the private subnet (as described before).

### Task 4: Create and Attach an Internet Gateway

- As per Experiment 5, Task 4.

### Task 5: Create Route Tables and Associate IGW

- For the public subnet, create a route to 0.0.0.0/0 via the IGW.

### Task 6: Create a NAT Gateway

1. **Allocate an Elastic IP:**
   - In the VPC console under **NAT Gateways**, click **Create NAT Gateway**.
   - Select the public subnet and allocate a new Elastic IP.
2. **Create NAT Gateway and Wait Until It’s Available.**

### Task 7: Update the Private Route Table

1. **Private Subnet Route Table:**
   - Add a route (0.0.0.0/0) that targets the NAT Gateway.
2. **Test Outbound Connectivity:**
   - SSH into the instance in the private subnet (via the public instance if needed) and verify internet access (e.g., `curl http://checkip.amazonaws.com`).

---

## Experiment 7: AWS S3 Bucket and Static Website Hosting

### Task 1: Create an S3 Bucket

1. **In the S3 Console:**
   - Click **Create Bucket**.
   - Choose a unique bucket name and a region.
   - Accept defaults or configure options as needed.

### Task 2: Upload Files (HTML, Images)

1. **Upload:**
   - In your bucket, click **Upload** and add your HTML files, images, and any other static assets.

### Task 3: Set Proper Permissions

1. **Bucket Policy and ACLs:**
   - Edit the bucket policy to allow public read if hosting a public website. For example:
     ```json
     {
       "Version": "2012-10-17",
       "Statement": [
         {
           "Sid": "PublicReadGetObject",
           "Effect": "Allow",
           "Principal": "*",
           "Action": "s3:GetObject",
           "Resource": "arn:aws:s3:::your-bucket-name/*"
         }
       ]
     }
     ```
   - Save the policy.
## Task 4: Enable Versioning & Cross-Region Replication
1. Versioning:
-Bucket → Properties → Versioning → Enable

-Now any new upload keeps the old versions too.

2**Cross-Region Replication:**
-Create another bucket in a different region.

-Source bucket → Management → Replication Rules

-Add destination bucket.

-Grant permissions as guided.

-Save rule → any new file is auto-copied to the other region.

✅ Task 5: Enable Static Website Hosting
1️⃣ Bucket → Properties → Static website hosting

Enable.

Index document: index.html

Error document: index.html (optional)

It will show a bucket website endpoint, e.g.:

arduino
Copy code
http://my-static-website-<yourname>.s3-website.ap-south-1.amazonaws.com
✅ Task 6: Test your website
Copy the endpoint URL


### Task 4: Enable Static Website Hosting

1. **Static Website Hosting:**
   - In your bucket properties, navigate to **Static website hosting**.
   - Enable it and set the index document (e.g., `index.html`) and error document as needed.
   - Note the endpoint URL provided; test it in your browser.

---

## Experiment 8: AWS Lambda Triggered by S3 Events to Update DynamoDB

### Task 1: Create a Lambda Function

1. **Create Lambda Function:**
   - In the AWS Lambda console, click **Create Function**.
   - Choose “Author from scratch” and provide a name (e.g., `S3toDynamoDBUpdater`).
   - Set a runtime (Node.js, Python, etc.) based on the code from `tinyurl.com/kmitdemocode` (review the code to ensure dependencies are met).
   - Assign an appropriate IAM role that allows access to S3, DynamoDB, and CloudWatch Logs.
2. **Upload Code:**
   - Paste or upload the Lambda function code.
   - Save and test using a sample event if available.

### Task 2: Set Up Event Trigger via S3 (API Gateway Optional)

1. **Direct S3 Trigger (Typical):**
   - In the Lambda configuration, add a trigger.
   - Choose **S3** as the source.
   - Specify the bucket and event type (e.g., “All object create events”).
2. **Using API Gateway (if required by the exercise):**
   - In the API Gateway console, create a new API.
   - Configure an endpoint that, when invoked, triggers your Lambda function.
   - In your S3 bucket, set up an event notification to call the API endpoint. *(Note: Typically S3 directly invokes Lambda. An API Gateway may be used if you have an intermediary layer.)*

### Task 3: Update DynamoDB from Lambda

1. **In the Lambda Code:**
   - Import the AWS SDK (if not already available).
   - Use the SDK to connect to your DynamoDB table.
   - Within the Lambda function’s logic, on receipt of an S3 event, update (or put) a record in the DynamoDB table.
   - For example (in Node.js):
     ```javascript
     const AWS = require('aws-sdk');
     const dynamoDB = new AWS.DynamoDB.DocumentClient();
     
     exports.handler = async (event) => {
       // Parse S3 event and extract necessary details
       const params = {
         TableName: 'YourDynamoDBTable',
         Item: {
           id: "UniqueIdentifier",  // e.g., S3 object key or generated ID
           timestamp: new Date().toISOString(),
           // other attributes as needed
         }
       };
       try {
         await dynamoDB.put(params).promise();
         return { statusCode: 200, body: JSON.stringify('Success') };
       } catch (err) {
         return { statusCode: 500, body: JSON.stringify(err) };
       }
     };
     ```
2. **Test the Integration:**
   - Upload an object to the specified S3 bucket.
   - Verify that the Lambda function is triggered.
   - Confirm that the record appears (or is updated) in the DynamoDB table.

### Experiment 9: Amazon Lex
 Step 1: Open Lex
Go to AWS Console → Amazon Lex V2

 Step 2: Create a Bot
1️ Click Create bot

2️⃣ Choose:

Bot name: AirlineBookingBot

Runtime: Lex V2

IAM role: Let Lex create one for you

Children: None

3️⃣ Language: English (US)

4️⃣ Session timeout: e.g. 5 min
5️⃣ Click Next, then Create

✅ Step 3: Add Intent
Click Intents → Add intent → Create new intent

Name: BookFlight

✅ Step 4: Add Slots
Slots = user inputs

Add:

Name	Slot Type	Prompt
DepartureCity	AMAZON.City	"Where are you flying from?"
Destination	AMAZON.City	"Where are you flying to?"
TravelDate	AMAZON.Date	"When are you travelling?"
Passengers	AMAZON.Number	"How many passengers?"
Class	AMAZON.FlightClass	"Which class do you prefer (Economy/Business)?"

✅ Slot order: make sure Passengers comes before Class so you can use Passengers to decide the Class.

✅ Step 5: Add Lambda (optional)
👉 If you want to process logic (like auto-upgrade to Business), you can connect a Lambda function.

Example:

If Passengers > 4 and user said Economy → override slot value to Business.

Or you can handle this in closing response using Conditional branching.

✅ Step 6: Closing Response
Add a response template:

Example:

yaml
Copy code
Booking Confirmed!  
{DepartureCity} → {Destination} | {TravelDate}  
{Passengers} Passengers | {Class} Class  
Meals: 2 Veg, 3 Non-Veg | Seats auto-selected  
Total: ₹87,500 (20% group discount applied)  
Frequent Flyer: GA-7X9B2P (Earned: 875 points)
✅ If you want dynamic logic, use Lambda to:

Check slots

Compute price

Format confirmation

✅ Step 7: Build and Test
1️⃣ Click Build to compile.

2️⃣ Click Test to launch chat:

Hi → bot responds.

Try:

css
Copy code
I want to book a flight  
From Delhi to Mumbai  
On Dec 25  
5 passengers  
Economy
Check that your closing message upgrades to Business Class if Passengers > 4.

✅ Step 8: Deploy
When ready, you can:

Connect to Amazon Connect (call center)

Add as a website chatbot (via Lex Web UI or Amazon Connect Contact Flow)


 ### Exp 10 DynamoDB
Manage a Products table for an e-commerce store:

Columns: ProductID (Partition Key), Name, Category, Price, StockCount

Insert 3 products:

One via Console Form (e.g., Smartphone)

One via JSON (e.g., Laptop)

One via PartiQL (e.g., Headphones)

Scan for all electronics

Query by ProductID

PartiQL for products under $50 with stock

✅ Step 1: Create the Table
1️⃣ Go to DynamoDB Console → Tables → Create Table

Table name: Products

Partition key: ProductID (String)

Leave Sort key blank

Provisioned or On-Demand: On-Demand (free tier friendly)

Create.

✅ Step 2: Insert Record via Console Form
1️⃣ Click Tables → Products → Explore Table Items

2️⃣ Click Create item

ProductID: P1001

Name: Smartphone

Category: Electronics

Price: 699

StockCount: 20

✅ Click Create item

✅ Step 3: Insert Record via JSON
1️⃣ Click Create item → Switch to JSON

2️⃣ Paste:

json
Copy code
{
  "ProductID": "P1002",
  "Name": "Laptop",
  "Category": "Electronics",
  "Price": 1200,
  "StockCount": 15
}
✅ Click Create item

✅ Step 4: Insert Record via PartiQL
1️⃣ Click PartiQL editor (in the left sidebar)

2️⃣ Run:

sql
Copy code
INSERT INTO "Products" VALUE {
  'ProductID': 'P1003',
  'Name': 'Headphones',
  'Category': 'Electronics',
  'Price': 45,
  'StockCount': 30
};
✅ Click Run

✅ Step 5: Scan to get all Electronics
sql
Copy code
SELECT * FROM "Products" WHERE "Category" = 'Electronics';
✅ Click Run — see all 3 items!

✅ Step 6: Query by ProductID
To get Laptop by ID:

sql
Copy code
SELECT * FROM "Products" WHERE "ProductID" = 'P1002';
✅ This returns Laptop.

✅ Step 7: PartiQL — Products under $50 with stock
sql
Copy code
SELECT * FROM "Products"
WHERE "Price" < 50 AND "StockCount" > 0;
✅ This finds Headphones (or any other cheap item).

⚡ ✅ Exp 10 done!
✔ Table created
✔ 3 items added — Console, JSON, PartiQL
✔ Scanned & queried
✔ ✅ You know how to read/write DynamoDB


 ### Exp 11 Goal
You’ll:
1️⃣ Create an IAM Group
2️⃣ Make a Custom Policy
3️⃣ Add Users to the Group
4️⃣ Create an IAM Role for EC2 with trust policy
5️⃣ Test that the user can launch EC2 and access the S3 bucket

✅ Step 1: Create IAM Group
1️⃣ Go to IAM Console → User Groups → Create Group
2️⃣ Name: WebDevTeam
3️⃣ No policies attached yet → click Create Group

✅ Step 2: Create Custom Managed Policy
1️⃣ IAM → Policies → Create policy → JSON

2️⃣ Paste:

json
Copy code
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "ec2:*",
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:PutObject",
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::dev-website-assets",
        "arn:aws:s3:::dev-website-assets/*"
      ]
    }
  ]
}
✅ Click Next → Name: WebDevPolicy → Create Policy

✅ Step 3: Attach Policy to Group
1️⃣ IAM → User Groups → WebDevTeam → Add permissions

2️⃣ Attach WebDevPolicy → Save.

✅ Step 4: Create IAM Users
1️⃣ IAM → Users → Add user

User: dev1

Access type: AWS Management Console access

Password: custom or autogenerated

2️⃣ Add to Group: WebDevTeam

3️⃣ Repeat for dev2.

✅ Step 5: Create IAM Role for EC2
1️⃣ IAM → Roles → Create role

Trusted entity: AWS service

Use case: EC2

2️⃣ Attach policy:

json
Copy code
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::dev-website-assets",
        "arn:aws:s3:::dev-website-assets/*"
      ]
    }
  ]
}
✅ Or attach AmazonS3ReadOnlyAccess if you prefer.

3️⃣ Name it: EC2WebServerRole

4️⃣ Create Role

✅ Step 6: Test
1️⃣ Sign in as dev1 in AWS Console → EC2 → Launch an instance.

2️⃣ During instance launch, in Advanced → IAM Role, attach EC2WebServerRole.

3️⃣ SSH into the instance → run:

bash
Copy code
aws s3 ls s3://dev-website-assets --region <your-region>
✅ Should list objects!

⚡ ✅ Exp 11 done!
✔ IAM Group (WebDevTeam)
✔ Custom Policy (WebDevPolicy)
✔ Users (dev1 & dev2)
✔ EC2 Role (EC2WebServerRole)
✔ Tested EC2 → S3 access

### Part A — SNS + SQS
✅ Step 1: Create SNS Topic
1️⃣ Go to SNS Console → Topics → Create Topic

Type: Standard

Name: order-events

✅ Click Create

✅ Step 2: Create SQS Queues
1️⃣ Go to SQS Console → Create Queue

Queue 1: payment-queue

Queue 2: inventory-queue

Leave as Standard queue

Create both

✅ Step 3: Subscribe SQS to SNS
1️⃣ SNS → Topics → order-events → Create subscription

Protocol: SQS

Endpoint: ARN of payment-queue

2️⃣ Repeat → add inventory-queue

✅ Now SNS → both SQS queues get same message!

✅ Step 4: Test
1️⃣ SNS → order-events → Publish message

Example JSON:

json
Copy code
{
  "order_id": "O123",
  "customer_id": "C456",
  "amount": 250
}
✅ Publish → both queues get it!

2️⃣ SQS → payment-queue → Poll for messages
✅ Should see the message.

Same for inventory-queue.

✅ Part B — CloudFront
✅ Step 1: Upload your static site to S3
1️⃣ Create bucket: blog-static-files
2️⃣ Upload index.html, styles.css, etc.

✅ Step 2: Make bucket public
Bucket → Permissions → Block Public Access → Uncheck all

Add Bucket Policy:

json
Copy code
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::blog-static-files/*"
    }
  ]
}
✅ Step 3: Create CloudFront Distribution
1️⃣ Go to CloudFront Console → Create Distribution

2️⃣ Origin Domain: pick your blog-static-files bucket

3️⃣ Set Default Root Object → index.html

4️⃣ Click Create Distribution

✅ You’ll get a CloudFront Domain Name → copy it.

✅ Step 4: Test
Open your CloudFront URL — you’ll see your static website loading worldwide, faster!




---

## Final Notes

- **IAM Roles and Permissions:**  
  Ensure that all your EC2 instances, Lambda functions, and other resources have the proper IAM roles and policies attached to perform the actions (accessing S3, DynamoDB, mounting EFS, etc.).

- **Security Group Settings:**  
  Adjust security groups cautiously—allow only the necessary inbound ports (SSH, RDP, HTTP, NFS, etc.) to ensure your environment is secure.

- **Region and Availability Zone:**  
  When attaching EBS volumes, EFS, or copying snapshots, verify that resources are in the correct region and availability zone.

- **Testing and Troubleshooting:**  
  For each experiment, test connectivity and functionality immediately after setup. Monitor CloudWatch logs (for Lambda or EC2) for any errors.

By following the steps above, you should be able to execute these experiments in AWS successfully. Be sure to consult the AWS documentation for further details or troubleshooting as needed.
