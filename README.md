# Centralized-Log-Server
![Rsyslog drawio](https://github.com/Akshaykumar05/Centralized-Log-Server/assets/114390890/3c25d30c-3efc-4b2d-a051-8a9d9d7d4dbc)


### Note: This document is classified and can be used only by Infra team of e-Transport project.
* In this task, I have to transfer logs of Tomcat Servers and HAProxy to a Centralised Log Server using **rsyslog** tool. So that Developer can have all the logs together to moniter.
  ### Steps
  * Create 2 VMs, one for Tomcat services and HAProxy and other for Centralised log server.
  * VM1- Client Server: Set-up two Tomcat services named Tomcat1, Tomcat2 and HAProxy.
  * VM2- Centralised Log Server: rsyslog
  * Configure rsyslog on Tomcat and haproxy to forward logs from VM1 to VM2.

 ### Centralised Log Server set up
1. Create a VM2 with atleast 2 GB RAM and 30 GB Hard disc space.
2. Install Ubuntu on it.
3. Install **rsyslog package**
   
   ```
   sudo apt update -y
   ```
   ```
   sudo apt install rsyslog -y
   ```
   ![rsyslog install](https://github.com/Akshaykumar05/NIC/assets/114390890/78f06ca3-7859-4e11-8ec9-a3b2783dc3ce)

4. Note: If you are using CentOS 8 / RHEL 9. then by default, Rsyslog  comes installed on it. To verify the status of Rsyslog, use the command:
   ```
   systemctl status rsyslog
   ```
   ![image](https://github.com/Akshaykumar05/NIC/assets/114390890/48cfdb3e-31f1-4881-92cb-6da939834106)

   
5. Next, we need to modify a few settings in Rsyslog configuration file. Edit the rsyslog configuration file:
   ```
   sudo vim /etc/rsyslog.conf
   ```
6. Sroll the file and uncomment the following lins to allow reception of logs via UDP and TCP protocol.
     
   ```
   module(load="imudp")
   input(type="imudp" port="514")

   module(load="imtcp")
   input(type="imtcp" port="514")
   ```
   ![rsyslog conf](https://github.com/Akshaykumar05/NIC/assets/114390890/4c1d71fc-5ebd-45f3-bbd5-56dd47c698c5)

   * Save and exit the configuration file.
  
7. To recieve the logs from the client server, we need to open Rsyslog default port 514 on the firewall. Run the following commands to achieve this:
   
   ```
   firewall-cmd  --permanent -add-port=514/tcp
   ```
   ```
   firewall-cmd  --permanent -add-port=514/udp
   ```
   * Next, reload the firewall to save the changes.

   ```
   sudo firewall-cmd --reload
   ```

   * Sample output
     
   ![firewall tcp,udp](https://github.com/Akshaykumar05/NIC/assets/114390890/922eb207-338e-4d72-9ee0-b92ebeda301c)


9. Set up a log file to store incoming logs:
   * Add the following line to the end of the file:
     
   ```
   *.* /var/log/remote.log
   ```
10. Restart the rsyslog service:
   ```
   sudo systemctl restart rsyslog
   ```
11. To enable Rsyslog on boot, run the following command:

    ```
    sudo systemctl enable rsyslog
    ```
12. Now, we need to confirm that the Rsyslog service is listening on port 514, use the **netstat** commands as follow:
    
    ```
    sudo netstat -tulnp
    ```
    * Sample output
      
    ![netstat 514,22](https://github.com/Akshaykumar05/NIC/assets/114390890/902adf16-f87b-4b49-a74a-5f25c3053f2a)

    * Perfect! we have successfully configured our Rsyslog server to recieve logs from the client system.
    * To view log messages in real-time run the command:

    ```
    sudo tail -f /var/log/remote.log
    ```

### Configuring the client servers on Ubuntu (VM1)
1. Set-up the Rsyslog server same as we did in above machine (VM2).
2. Check the status by runnibg the command:
   
   ```
   sudo systemctl status rsyslog
   ```
   * Sample output
  
3. Next, proceed to open the rsyslog configuration file

   ```
   sudo vim /etc/rsyslog.conf
   ```
4. At the end of the file, append the following line.
   * Uncomment the following lines in the conf file:

   ![rsyslog conf tcp,udp](https://github.com/Akshaykumar05/NIC/assets/114390890/3a59e37a-960e-4a46-b1c9-9a69096f1af6)

   * Then, at the end of the file, append the following line:

   ![rsyslog conf VM1](https://github.com/Akshaykumar05/NIC/assets/114390890/ae51ecfa-73de-4edb-bbec-7129c24a346f)

 5. Save and exit the configuration file. Same as the Rsyslog server, open the port 514 which is the default Rsyslog port on the firewall.

   ```
   firewall-cmd  --permanent -add-port=514/tcp
   ```
   ```
   firewall-cmd  --permanent -add-port=514/udp
   ```
   * Now, reload the firewall to save the changes.

   ```
   sudo firewall-cmd --reload
   ```
6. Next, restsrt th rsyslog service and enable Rsyslog on boot with the following command:

   ```
   sudo systemctl restart rsyslog
   ```
   ```
    sudo systemctl enable rsyslog
   ```

7. Testing the logging operation
   * Having successful set up and configured Rsyslog and client servers, it's time to verify the configurations are working intended.
   * We'll hit the following command on VM1 (client server)

   ```
   # logger "Hello folks! This is our log testing"
   ```
   * Now head out to the Rsyslog server and run the command below to check the transfered logs in real-time.

   ```
   sudo tail -f /var/log/remote.log
   ```

   * The output from the command run on the client system should register on the Rsyslog servers's log messages to imply that the Rsyslog server is now recieving logs from the client servers.
     
    ![log testing](https://github.com/Akshaykumar05/Centralized-Log-Server/assets/114390890/79f6c61e-0894-47f5-9dbe-99f8813f5c8c)

   * And that's it, guys! We have successfully setup the Rsyslog server to recieve log messages from a client server.
     
     ![image](https://github.com/Akshaykumar05/Centralized-Log-Server/assets/114390890/33f4a5c1-7dfb-4ba2-adb4-97d4c9543568)

-------------------------------
-------------------------------
## Live projects case studies:
* There is a requirement of log of 1 day from a given server. (Need logs of 11 Feb of eDetection application with server IP 10.242.36.130)
### Steps:
1. Login to the required server and create a folder with the same date to save the log files.
   ```
   mkdir /tmp/eDetection_log_11_feb
   ```
   ![log-file1](https://github.com/user-attachments/assets/740dcb13-2ff9-4d6f-ae75-86d48d10e84a)

2. Check the directory
   ```
   ls -lrth
   ```
   ![log-file2](https://github.com/user-attachments/assets/bed3bcbe-54ea-4be7-8219-da5dd6e20043)

3. Now go to the path of logs
   ```
   /u01/log/app_log
   ```
   ```
   ls -lrth
   ```
   ![log-file-3](https://github.com/user-attachments/assets/9b85cf86-2c6f-4c62-bf19-679d8837149a)
   
 4. Copy the log of 11 feb in the created folder in tmp.
    ```
    cp eDetection.log.2025-02-11.* /tmp/eDetection_log_11_feb
    ```
    ![log-file-4](https://github.com/user-attachments/assets/b21d42fb-085c-49e5-a6c6-ec4ab0752595)

 5. Again check either log are copied
    ```
    cd eDetection_log_11_feb
    ```
    ```
    cd
    ```
    ![log-file-5](https://github.com/user-attachments/assets/0da1b3e9-5444-425d-97dd-4b521b731bd2)

    
 6. Now zip the directory
    ```
    zip -r eDetection_log_11_feb.zip eDetection_log_11_feb
    ```
    ![log-file-6](https://github.com/user-attachments/assets/79407005-b448-4e4b-acf3-ea4036ed34f5)


 7. At last copy the zip file in the local.
    ```
    scp -r etrans-infra-mon10@10.242.36.130:/tmp/eDetection_log_11_feb.zip .
    ```
 * The job is done, now you can send the zip file to the Developer.
   
----------------------------------------------------------------------------------
## On client server
Check the details of Tomcat running on the application, and configure them so that logs can be transferred on the given URL.

    ![log-file-7](https://github.com/user-attachments/assets/ec57f8bc-6e69-4397-9c1f-8233ce3dddd9)

 

-----------------------------------------------
    
  


