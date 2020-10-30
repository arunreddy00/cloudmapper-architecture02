CloudMapper which allows you to achieve all the above within your AWS environment as well as a recent feature that allows it to be a continuous monitoring and auditing solution. Furthermore providing you comply with their licence, it is free. 

This guide shows you how to setup the Duo CloudMapper with the demo data then link it into your AWS environment.

For this guide I am using a fresh Amazon EC2 instance with a security group that allows TCP 8000 and SSH through from my IP only.

Once the EC2 is built ensure it is up to date

sudo yum update

For AWS Linux, Centos, Fedora, RedHat install the following

sudo yum install autoconf automake libtool python3-devel.x86_64 python3-tkinter python-pip jq awscli pipenv git

Clone the CloudMapper GitHub repository

git clone https://github.com/duo-labs/cloudmapper.git

Change directory and install

cd cloudmapper/
pipenv install --skip-lock
pipenv shell

Demo data is available that can be generated using the following commands then accessed using http://127.0.0.1:8000

# Generate the data for the network map
python cloudmapper.py prepare --config config.json.demo --account demo

# Generate a report
python cloudmapper.py report --config config.json.demo --account demo

# Initialise the webserver
python cloudmapper.py webserver

If you want to access this data externally use the --public switch and ensure any firewalls/security groups are allowed

python cloudmapper.py webserver --public

The demo data should now be available externally using http://<ip>:8000

Sample report data is available on http://<ip>:8000/account-data/report.html or see https://duo-labs.github.io/cloudmapper/account-data/report.html

Once you have the demo data working correctly you now need to link it into your AWS account to generate real data.

Firstly from the cloudmapper directory, copy the demo json file

cp config.json.demo config.json

Edit the json file updating the account ID and name with your details and the CIDR blocks from your VPC's

vi config.json

Exit Esc and save the config.json file :wq!

You now need to setup an account with appropriate permissions to collect data from your AWS
Create an IAM User with the necessary permissions

Sign into the AWS Management Console and open the IAM console at https://console.aws.amazon.com/iam/

Select Users then Add User, give it a logical name and select programmatic access

On the permissions page, select attach existing policies directly, then search and select

ViewOnlyAccess

SecurityAudit

add any tag and on the review page it should look like

Once reviewed select create user and download the access keys.

Add the access keys to a CLI profile using the aws configure command to create a profile (This was installed with the awscli package earlier)

 

Once the credential file has been added, run the following command replacing the account name and profile credentials which will start the collection

python cloudmapper.py collect --account infraengineer --profile default

Once the data has been collected, it needs prepared, reported on and presented through the web server

# Generate the data for the network map
python cloudmapper.py prepare --config config.json --account InfraEngineer

# Generate a report
python cloudmapper.py report --config config.json --account InfraEngineer

# Initialise the webserver
python cloudmapper.py webserver --public

Security group permitting your map should now be available using http://<ip>:8000 and report data on http://<ip>:8000/account-data/report.html

The example image below is my lab environment but in a larger environment this picture would have a lot more assets and security groups to visualise and review.

Now you have the visibility of your estate, you can start using it as part your auditing, security analysis
