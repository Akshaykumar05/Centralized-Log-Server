# Centralized-Log-Server
 ![image](https://github.com/Akshaykumar05/NIC/assets/114390890/a4b724ce-dd56-47cb-afee-49cf9b9f6cff)

* In this task, I have to transfer logs of Tomcat Servers and HAProxy to a Centralised Log Server using **rsyslog** tool. So that Developer can have all the logs together to moniter.
  ### Steps
  * Create 2 VMs, one for Tomcat services and HAProxy and other for Centalised log server.
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


8. Set up a log file to store incoming logs:
   * Add the following line to the end of the file:
     
   ```
   *.* /var/log/remote.log
   ```
9. Restart the rsyslog service:
   ```
   sudo systemctl restart rsyslog
   ```
10. To enable Rsyslog on boot, run the following command:

    ```
    sudo systemctl enable rsyslog
    ```
11. Now, we need to confirm that the Rsyslog service is listening on port 514, use the **netstat** commands as follow:
    
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

