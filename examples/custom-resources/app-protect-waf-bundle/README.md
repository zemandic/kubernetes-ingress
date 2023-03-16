# WAF

In this example we deploy the NGINX Plus Ingress Controller with [NGINX App Protect](https://www.nginx.com/products/nginx-app-protect/), a web application and then configure load balancing and WAF protection for that application using the VirtualServer resource.

## Prerequisites

1. Follow the installation [instructions](https://docs.nginx.com/nginx-ingress-controller/installation) to deploy the Ingress Controller (including persisted volume PV) with NGINX App Protect. NGINX Ingress Controller deployment should include mounted volume (persisted volume PV). The WAF policy bundle file (```dataguard-alarm.tgz```) should be on the volume.
1. Save the public IP address of the Ingress Controller into a shell variable:
    ```
    $ IC_IP=XXX.YYY.ZZZ.III
    ```
1. Save the HTTP port of the Ingress Controller into a shell variable:
    ```
    $ IC_HTTP_PORT=<port number>
    ```

## Step 1. Deploy a Web Application

Create the application deployment and service:
```
$ kubectl apply -f webapp.yaml
```

## Step 2 - Configure the WAF Policy (Policy Bundle)

Note: The policy bundle ```dataguard-alarm.tgz``` should already exist in the mounted volume in the NGINX Ingress Controller deployment.


1. Configure the WAF bundle
    ```
    $ kubectl apply -f waf.yaml
    ```

## Step 3 - Configure Load Balancing

1. Create the VirtualServer Resource:
    ```
    $ kubectl apply -f virtual-server.yaml
    ```

Note that the VirtualServer references the policy bundle `waf-bundle` referenced in Step 2.

## Step 4 - Test the Application

To access the application, curl the coffee and the tea services. We'll use the --resolve option to set the Host header of a request with `webapp.example.com`

1. Send a request to the application:
    ```
    $ curl --resolve webapp.example.com:$IC_HTTP_PORT:$IC_IP http://webapp.example.com:$IC_HTTP_PORT/
    Server address: 10.12.0.18:80
    Server name: webapp-7586895968-r26zn
    ...
    ```

1. Now, let's try to send a request with a suspicious URL:
    ```
    $ curl --resolve webapp.example.com:$IC_HTTP_PORT:$IC_IP "http://webapp.example.com:$IC_HTTP_PORT/<script>"
    <html><head><title>Request Rejected</title></head><body>
    ...
    ```
1. Lastly, let's try to send some suspicious data that matches the user defined signature.
    ```
    $ curl --resolve webapp.example.com:$IC_HTTP_PORT:$IC_IP -X POST -d "apple" http://webapp.example.com:$IC_HTTP_PORT/
    <html><head><title>Request Rejected</title></head><body>
    ...
    ```
    As you can see, the suspicious requests were blocked by App Protect

1. To check the security logs in the syslog pod:
    ```
    $ kubectl exec -it <SYSLOG_POD> -- cat /var/log/messages
    ```
