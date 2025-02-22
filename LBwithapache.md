# Setting Up an Apache Load Balancer on AWS with Ubuntu 24.04

This documentation will guide you through the process of how i was able to setup an Apache load balancer on a new EC2 instance running Ubuntu 24.04 to distribute traffic between my two web servers. In the previous documentation , i already configured the 3 tier architecture with 2 webservers

## Prerequisites

- AWS account
- Existing 3-tier architecture with two web servers
- Apache installed on both web servers
- EC2 instance for the load balancer, yo can name it **project-8-apache-lb**
- Set up security group and allow port 80 to be accessed from anywhere **0.0.0.0/0**

![load balancer](/images/lb%20instance.png)


## Step 1: Install and Configure Apache on the Load Balancer Instance

1. **SSH into the Load Balancer Instance**:
    - Use your terminal or an SSH client to connect to your instance:
      ```bash
      ssh -i /path/to/your-key.pem ubuntu@your-load-balancer-public-ip
      ```



2. **Install Apache**:
    - Update the package index and install Apache:
      ```bash
      sudo apt update -y
      sudo apt install apache2 -y
      ```

![Install Apache](/images/installed%20apache.png)

3. **Enable and Start Apache**:
    - Enable Apache to start on boot and then start the service:
      ```bash
      sudo systemctl enable apache2
      sudo systemctl start apache2
      ```


4. **Configure Apache for Load Balancing**:
    - Install the necessary Apache modules:
      ```bash
         #Install apache2
        sudo apt update
        sudo apt install apache2 -y
        sudo apt-get install libxml2-dev

        #Enable following modules:
        sudo a2enmod rewrite
        sudo a2enmod proxy
        sudo a2enmod proxy_balancer
        sudo a2enmod proxy_http
        sudo a2enmod headers
        sudo a2enmod lbmethod_bytraffic
        
        #Restart apache2 service
        sudo systemctl restart apache2

      ```

    - Open the Apache configuration file:
      ```bash
      sudo vi /etc/apache2/sites-available/000-default.conf
      ```

    - Add the following configuration to set up load balancing:
      ```apache
      <Proxy "balancer://mycluster">
         BalancerMember http://webserver1-private-ip
         BalancerMember http://webserver2-private-ip
         ProxySet lbmethod=byrequests
      </Proxy>

      ProxyPass "/" "balancer://mycluster/"
      ProxyPassReverse "/" "balancer://mycluster/"
      ```

    - Replace `webserver1-private-ip` and `webserver2-private-ip` with the private IP addresses of your two web servers.
    - Save and close the file.
![permissions](/images/installed%20LB%20and%20configured%20permisiions.png)
![config](/images/configuring%20LB.png)


5. **Restart Apache**:
    - Restart Apache to apply the changes:
      ```bash
      sudo systemctl restart apache2
      ```

6.  **Unmount **/var/log/httpd/++ and ensure each webservers has thier own log directory  **:
   
      ```bash
      sudo umount /var/log/httpd
      ```
     - Refresh the load balancer ip address in order to observe the different requests sent by the browser and to which webservers: use the command below to view the log directory on each webserver
     ```bash
      sudo tail -f /var/log/httpd/access_log
      ```
![log](/images/logged%20on%20both%20servers.png)   


## Step 4: Verify the Load Balancer Setup

1. **Access the Load Balancer**:
    - Open a web browser and navigate to the public IP address of your load balancer instance.
    - You should see the content served by your web servers.

![Access Load Balancer](/images/lb%20successfully%20configured.png)
![Access Load Balancer](/images/logged%20in.png)



---

we have successfully set up an Apache load balancer on AWS to distribute traffic between our two web servers.

## However, we can setup local dns name resolution in order to be able to identify between webservers, this can be done through :

1. **Using the /etc/hosts/ file**:
    - open the /etc/hosts/ file
         ```bash
      sudo vi /etc/hosts
      ```
    - Add the following lines
       ```bash
      webserver1-ipaddress web1
      webserver2-ipaddress web2 
      ```
    - Save and exit.
  
    ![etc/hosts](/images/etchosts.png)
  
      
2. **Go back to apache config**:
      ```bash
      sudo nano /etc/apache2/sites-available/000-default.conf
      ```
    - update the webservers ipaddress with the local dns name , that is :
        ```bash
         BalancerMember http://webserver1-private-ip  to  BalancerMember http://web1
         BalancerMember http://webserver2-private-ip  to  BalancerMember http://web2
         ```
    - Save and exit.
    - You can use curl to see if it works. P.S, the dns name was set locally and as such cannot be accessed from the internet or from another server
       ```bash
            curl http://web1
            curl http://web2
         ```
    ![curl](/images/using%20curl.png)
        
