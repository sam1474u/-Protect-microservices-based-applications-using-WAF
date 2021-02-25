# Protect-microservices-based-applications-using-WAF

Prepare Your Environment to Use Web Application Firewall
Prepare your environment to be able to use the Web Application Firewall (WAF) service of Oracle Cloud Infrastructure to secure your application.

## Before You Begin
**prerequisite**: Setup <a href="https://github.com/sam1474u/Deploy-Helidon-Based-Application-in-Kubernetes-Cluster">Helidon App</a>

Before creating the WAF policy, you need to know the public IP address of the employee-helidon-lb load balancer already been deployed. See Deploy a microservices-based RESTful Java application to Oracle Cloud.
Obtain and note the EXTERNAL-IP address of the load balancer of the sample Employee application. This will be required to create the WAF policy later.

      Copykubectl get service employee-helidon-lb

The preceding command should return an output similar to the following:

      CopyNAME                  TYPE           CLUSTER-IP     EXTERNAL-IP      PORT(S)        AGE
      employee-helidon-lb       LoadBalancer   10.X.X.X       132.X.X.X        80:31437/TCP   5m

EXTERNAL-IP displayed by Kubernetes is the same as the public IP address, which is displayed when you navigate to Core > Networking > Load Balancers in the Oracle Cloud Infrastructure console.

## Create a WAF Policy
To secure your application using WAF, first, you need to create a WAF policy.
<br/>
      1. Sign in to the Oracle Cloud Infrastructure console.<br/>
      2. Open the navigation menu. Under Governance and Administration, go to Security and click WAF Policies.<br/>
      3. If prompted, pick a compartment where the WAF policy should be created.<br/>
      4. Click Create WAF Policy.<br/>
      In the Create WAF Policy dialog box, enter the fields as follows:<br/>

   ![image](https://user-images.githubusercontent.com/42166489/109104885-d9cc4a80-7752-11eb-8099-fe04b3e3cf7e.png)

   ![image](https://user-images.githubusercontent.com/42166489/109104743-8f4ace00-7752-11eb-9be7-1d20cfaa41fb.png)<br/>


5. Click Create WAF Policy to close the dialog box.<br/>
6. Once created, load the details for Employee Demo Policy from the list of policies.<br/>

## Route Application Traffic Using WAF
To route traffic through WAF, the specified primary domain and any additional domains must resolve to the **CNAME Target** listed in the WAF policy information. This is presented as a notice when the policy is first created.

If the application is actually configured in a DNS zone, you would normally create a **CNAME or ALIAS** record for **employee.example.com** using **WAF_Target** as its value.

However, since this solution uses an **example.com** host name for the sample Employee application, you need to configure your machine (from which you'll access the application) to resolve the host.
<br/>

1. Set a WAF_TARGET environment variable for use in other commands, ensuring that you specify the CNAME Target listed in the WAF policy details as the value of the variable.<br/>
            # Mac or Linux
            export WAF_TARGET=employee-example-com.b.waas.oci.oraclecloud.net
            # Windows
            set WAF_TARGET=employee-example-com.b.waas.oci.oraclecloud.net
 2. Verify that traffic is flowing through WAF. Inspect the HTTP Response headers and expect to see the following: <br/>
      Server: ZENEDGE <br/>
      X-Cdn: Served-By-Zenedge <br/>

            curl -I-X GET http://${WAF_TARGET}/public/ -H'host: employee.example.com'
The preceding code should return an output similar to the following:

            HTTP/1.1 200 OK
            Content-Type: text/html
            Connection: keep-alive
            Server: ZENEDGE
            ETag: "1565120174000"
            X-Cache-Status: NOTCACHED
            Content-Length: 14258
            Date: Wed, 07 Aug 2019 20:22:35 GMT
            Last-Modified: Tue, 07 Aug 2019 19:36:14 GMT
            X-Zen-Fury: 1053bc526e3d722eadb457f2
            
3. Configure your machine to resolve the sample Employee application URL when loaded in a browser. This is done by creating an entry in your hosts file, which bypasses DNS and routes requests from your machine to the WAF IP address for **employee.example.com.**<br/><br/>
      Identify a (there may be several) network IP address for the **$WAF_TARGET.**
      
              nslookup $WAF_TARGET
      The preceding code should return an output similar to the following:
      
            Server:         192.168.X.X
            Address:        192.168.X.X#YY
            Non-authoritative answer:
            employee-example-com.b.waas.oci.oraclecloud.net canonical name = oci.tm.u.waas.oci.oraclecloud.net.
            oci.tm.u.waas.oci.oraclecloud.net  canonical name = us-seattle.u.waas.oci.oraclecloud.net.
            Name:   us-seattle.u.waas.oci.oraclecloud.net
            Address: 152.X.X.X
            Name:   us-seattle.u.waas.oci.oraclecloud.net
            Address: 152.X.X.Y
            Name:   us-seattle.u.waas.oci.oraclecloud.net
            Address: 152.X.X.Z
        
      Select any single address from the Non-authoritative answer section and create a hosts entry for the example primary domain in the /etc/hosts file as the following:

            # hosts file entry
            # <WAF IP Address>    <domain>  
               152.X.X.Z          employee.example.com

      ![image](https://user-images.githubusercontent.com/42166489/109106227-5fe99080-7755-11eb-869f-afcb26ae21ff.png)
      
      Open http://<employee.example.com>/public/ in a browser.
  4. The application now runs over WAF.




