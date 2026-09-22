To Deploy Spring boot web application in Cloud
create a simple hello world program
Generate the JAR of Your Springboot App
Java -jar app.jar 

With Out Docker
Select The Region 
Create a Virtual Machine
EC2- Elastic Compute Cloud -> 
Elastic:Scale Up and Scale Down
Compute: Provide cpu,ram ,Storage to execute the task

Create a EC2 Instances
Go to services-EC2
Launch on Instances
Give Name
os:select the os images like windows, ubuntu and you can also give your AmI(Amazon Machine Image)
select the instance type : computing things stoarge , cpu, ram
key pair: authentication for system: create a Key Pair pem or ppk file
network settings: security Groups -> security to the your instance,firewall
using ssh we can connect
Lunch Instance
ssh -> pem file should be read only
chmod 400 hello-world.pem
ssh pemfile user@IP
You logged on your system
Run Your Jar on EC2 
copy your local jar into virtual Machine
scp pem jarfile ec2-user@ip:/user/home/opt/production/

install the java version
sudo yum install java
sudo dnf install java

java -version
java -jar hello-word.jar

to acces the Applictaion: 
set up the security group
add inbound rule for http
------------------
using docker

install docker on ec2
sudo yum install docker
docker version
for docker daemon 
systemctl status docker.service
dockerize your springboot application



