# Web-Server-with-Strong-Firewall

# The Project- 
I want to create a Web Portal with full security. For this, I have made a setup for Wordpress Software with a dedicated Database Server so that only it will accessible by the wordpress not by the outside world. 

# We need to know the Wordpress- 
WordPress (WP, WordPress.org) is a free and open-source content management system (CMS) written in PHP and paired with a MySQL or MariaDB database. Features include a plugin architecture and a template system, referred to within WordPress as Themes. WordPress was originally created as a blog-publishing system but has evolved to support other types of web content including more traditional mailing lists and forums, media galleries, membership sites, learning management systems (LMS) and online stores. WordPress is used by more than 60 million websites, including 33.6% of the top 10 million websites as of April 2019, WordPress is one of the most popular content management system solutions in use. WordPress has also been used for other application domains such as pervasive display systems (PDS).

# Description and Contents- 

**Steps that I follow in this project:-**

**1.** _I write a Infrastructure using terraform in code that creates a VPC(Virtual Private Cloud)._ 

**2.** _In above VPC I have to create 2 subnets: Firstly, the public subnet [that accessible for Public World! ]. Second, the private subnet [ that Restricted for Public World! ]._

**3.** _Now, I create a internet gateway for public facing that connects my VPC/Network to the outside world i.e. internet and I attach this gateway to my VPC._

**4.** _Then, I create a routing table for the above Internet gateway so that my instance can able to connect to outside world, update it and associate it with the public subnet._

**5.** _Now, I launch an EC2 instance that has Wordpress already setup and configured, and having the security group that allows the port 80 so that our clients can connect to our wordpress site. I also attached the key to the instance to facilitate further credentials and logins._

**6.** _Again, launching one more EC2 instance that has MYSQL already setup and configured with security group allowing port 3306 in private subnet so that our wordpress VM can connect with the same. Also attach the key with the same._

**Note:-** _Wordpress instance has to be part of public subnet so that our client can connect with our site._ 

_MYSQL instance has to be part of private subnet so that outside world can't connect to it._ 

_Make sure that auto ip assign and auto dns name assignment option should be enabled._

# Procedure :-

**Step=>1**  First of all, I have to configure my AWS profile in my local system using cmd. Filling all credentials & press Enter.

              aws configure --profile Vishnu
              AWS Access Key ID [****************BXEO]:
              AWS Secret Access Key [****************Jt5v]:
              Default region name [ap-south-1]:
              Default output format [json]:
              
**Step=>2**  Now, I need to create a VPC. Amazon Virtual Private Cloud (Amazon VPC) a logically isolated section of the AWS Cloud where you can launch AWS resources in a virtual network that you created. You have full control over your virtual networking environment, in this, selection of your own IP address range, creation of subnets, and configuration of route tables and network gateways are there. You can use both IPv4 and IPv6 in your VPC for secure and easy access to resources and applications.        


The code using terraform to create a VPC is here:-

                resource "aws_vpc" "vishnu_vpc" {
                            cidr_block = "192.168.0.0/16"
                            instance_tenancy = "default"
                            enable_dns_hostnames = true
                            tags = {
                              Name = "vishnu_vpc"
                            }
                          }
                          
                         
**Step=>3**  Now, I need to create two subnets in this VPC :- 

**=>**  the public subnet [ Accessible for Public World! ]

**=>**  the private subnet [ Restricted for Public World! ]

Subnet is “part of the network”, in other words, part of entire availability zone. Each subnet must reside entirely within one Availability Zone and cannot span zones. The terraform code to create both the Private & the Public Subnet is as follows :

            resource "aws_subnet" "vishnu_public_subnet" {
                        vpc_id = "${aws_vpc.vishnu_vpc.id}"
                        cidr_block = "192.168.0.0/24"
                        availability_zone = "ap-south-1a"
                        map_public_ip_on_launch = "true"
                        tags = {
                          Name = "vishnu_public_subnet"
                        }
                      }
          
          
            resource "aws_subnet" "vishnu_private_subnet" {
                        vpc_id = "${aws_vpc.vishnu_vpc.id}"
                        cidr_block = "192.168.1.0/24"
                        availability_zone = "ap-south-1a"
                        tags = {
                          Name = "vishnu_private_subnet"
                        }
                      }
                      
                      
**Step=>4** Now, I create a Public facing Internet Gateway.

An internet gateway is a horizontally scaled, redundant, and highly available VPC component that allows communication between your VPC and the internet.

The code using terraform to create a VPC is here:-


          resource "aws_internet_gateway" "vishnu_gw" {
                      vpc_id = "${aws_vpc.vishnu_vpc.id}"
                      tags = {
                        Name = "vishnu_gw"
                      }
                    }
                    
                    
**Step=>5** Now, I create a Routing Table & associate it with the Public Subnet.

A routing table, or routing information base (RIB), is an electronic file or database-type object that is stored in a router or a networked computer, holding the routes (and in some cases, metrics associated with those routes) to particular network destinations.

This information contains the topology of the network close to it. 

The code using terraform to create a VPC is here:-

          resource "aws_route_table" "vishnu_rt" {
                      vpc_id = "${aws_vpc.vishnu_vpc.id}"

                      route {
                        cidr_block = "0.0.0.0/0"
                        gateway_id = "${aws_internet_gateway.vishnu_gw.id}"
                      }

                      tags = {
                        Name = "vishnu_rt"
                      }
                    }

          resource "aws_route_table_association" "vishnu_rta" {
                      subnet_id = "${aws_subnet.vishnu_public_subnet.id}"
                      route_table_id = "${aws_route_table.vishnu_rt.id}"
                    }
                    
                    
**Step=>6**  Now, I create the security group which works while launching Wordpress. 

This security group has the permissions for outside connectivity. 

A security group acts as a virtual firewall for your EC2 instances to control incoming and outgoing traffic. 

Inbound rules control the incoming traffic to your instance, and outbound rules control the outgoing traffic from your instance. 

Security groups are associated with network interfaces.


          resource "aws_security_group" "vishnu_sg" {

                      name        = "vishnu_sg"
                      vpc_id      = "${aws_vpc.vishnu_vpc.id}"


                      ingress {

                        description = "allow_http"
                        from_port   = 80
                        to_port     = 80
                        protocol    = "tcp"
                        cidr_blocks = [ "0.0.0.0/0"]

                      }


                         ingress {

                         description = "allow_ssh"
                         from_port   = 22
                         to_port     = 22
                         protocol    = "tcp"
                         cidr_blocks = ["0.0.0.0/0"]
                       }

                       ingress {

                         description = "allow_icmp"
                         from_port   = 0
                         to_port     = 0
                         protocol    = "tcp"
                         cidr_blocks = ["0.0.0.0/0"]
                       }

                       ingress {

                         description = "allow_mysql"
                         from_port   = 3306
                         to_port     = 3306
                         protocol    = "tcp"
                         cidr_blocks = ["0.0.0.0/0"]
                       }

                       egress {
                         from_port   = 0
                         to_port     = 0
                         protocol    = "-1"
                         cidr_blocks = ["0.0.0.0/0"]
                       }


                        tags = {

                        Name = "vishnu_sg"
                      }
                    }
                    
                    
**Step=>7**  Next, I create another security group which will be used to launch MySQL. 

This security group will keep the MySQL accessible only through the wordpress and not for outside world.


            resource "aws_security_group" "vishnu_sg_private" {

                        name        = "vishnu_sg_private"
                        vpc_id      = "${aws_vpc.vishnu_vpc.id}"

                        ingress {

                         description = "allow_mysql"
                         from_port   = 3306
                         to_port     = 3306
                         protocol    = "tcp"
                         security_groups = [aws_security_group.vishnu_sg.id]

                       }


                       ingress {

                         description = "allow_icmp"
                         from_port   = -1
                         to_port     = -1
                         protocol    = "icmp"
                         security_groups = [aws_security_group.vishnu_sg.id]

                       }

                       egress {

                        from_port   = 0
                        to_port     = 0
                        protocol    = "-1"
                        cidr_blocks = ["0.0.0.0/0"]
                        ipv6_cidr_blocks =  ["::/0"]
                      }


                      tags = {

                          Name = "vishnu_sg_private"
                        }
                      }
                      
                      
**Step=>8**  Now, we are good to go. 

We are going to launch our Wordpress and MySQL instances using all the resources that we have created above.

**Wordpress -**


        resource "aws_instance" "wordpress" {

                    ami           = "ami-ff82f990"
                    instance_type = "t2.micro"
                    key_name      =  "vishnu_key"
                    subnet_id     = "${aws_subnet.vishnu_public_subnet.id}"
                    security_groups = ["${aws_security_group.vishnu_sg.id}"]
                    associate_public_ip_address = true
                    availability_zone = "ap-south-1a"


                    tags = {
                      Name = "vishnu_wordpress"
                      }
                    }  
                    
                    
**MySQL -**

              resource "aws_instance" "sql" {
                          ami             =  "ami-08706cb5f68222d09"
                          instance_type   =  "t2.micro"
                          key_name        =  "vishnu_key"
                          subnet_id     = "${aws_subnet.vishnu_private_subnet.id}"
                          availability_zone = "ap-south-1a"
                          security_groups = ["${aws_security_group.vishnu_sg_private.id}"]

                          tags = {
                           Name = "vishnu_sql"
                           }
                         }       
                         
**Step=>9**  Now, we have to run our terraform code. 

For doing so, we have to run the **terraform init** command first.

This will download the necessary plugins.

![step 9](https://user-images.githubusercontent.com/66829650/90485537-44258200-e155-11ea-8688-d5403c970eab.png)


Then, we have to run the command **terraform apply --auto-approve.** 

This will run the code and create the mentioned resources on the configured AWS Cloud.

![step 9-part](https://user-images.githubusercontent.com/66829650/90486066-0b39dd00-e156-11ea-8f79-89bb466454a9.png)


**Now, we have to see our AWS dashboard & check that our Wordpress & MYSQL are running in the EC2 section.**
![step 9 part 2](https://user-images.githubusercontent.com/66829650/90486257-53f19600-e156-11ea-9c8d-fea5d934d9ca.png)


**We can access our Wordpress site using the Public IP address that is mentioned in the instance description.**
![step 9 launch](https://user-images.githubusercontent.com/66829650/90486459-961ad780-e156-11ea-9ac4-a67e4039cf65.png)

Kudos

!! We did it successfully !!

Finally, we launched a Web Portal for a company with a dedicated Database Server that can be accessed only by the Wordpress, facilitating the security of our content.

![PinkLogoWhiteWebsite](https://user-images.githubusercontent.com/66829650/90487070-8f409480-e157-11ea-9cfa-42c1968b423c.png)




