## Answers for datadog Solution engineer


# Setup the environment 

I create virtual machine  **Ubuntu/xenial** on **VirtualBox** using **Vagrant**  an I used datadog agent V6.

## Installing the agent
to install the agent on a ubuntu machine i run :
```bash
DD_API_KEY=<my_api_Key> bash -c "$(curl -L https://raw.githubusercontent.com/DataDog/datadog-agent/master/cmd/agent/install_script.sh)"
```

# Collecting Metrics:

 

 1. Adding the tag  edit the file /etc/datadog-agent/datadog.yaml :
 ``` /etc/datadog-agent/datadog.yaml 
 tags:
   - mytag:myubuntu-agent-onmac
  ```
  

  ![enter image description here](https://s3.eu-west-1.amazonaws.com/marouane/datadog-screenshot/Screen%20Shot%202018-08-08%20at%2015.24.53.png?response-content-disposition=inline&X-Amz-Security-Token=AgoGb3JpZ2luEAkaDGV1LWNlbnRyYWwtMSKAAhS2jf2Dl7tHi78yWig86mSfeWXVrdzeCn%2bwvNssGSsFFuIDt65q0KPHVctdceb%2bgbgHvE0xLSg1XcPTteIqKdvdR8N6BEHrLPfZKFbCbrAlh3c8OSE8ri%2bBst5uhKidZbFlUBYJucwRZskm8m/5tvCNTZ36oev1fjjKXDuaXLFr6VTlZfpdGG6YPQiDkfSdfcOqJU7EzNKYBxdHqgw3kXjhTfDxnPYcAiM3FIK5MOxRF64%2brgZ3Fdy9CWJMagBtsNags4ojyphGW08eq0KEp8obYD0h/o8sPU/Bgdau2NI%2biQtScpLklldFobYMVucEpWm3AhUDGw0ocpKFdcTd/Jcq7AII7///////////ARABGgwyOTk1MjIwNTQ2OTgiDMXzAcwbjdjmowmYgSrAAhcbU3pmRtisjJmHevBe9j0S4CpxZOjYvoLz54G1tgcKab%2bRvh4AvpQH8gEjyuupeK65Y0vbdpPUPreD0IoCDrxKOabr2rxeFPCvpiUbqXEZ93QFYh/tp06Tf%2bfS26vVv65yCq1J6CDbI3fCzhJLR%2bC0fmZRtn2kaoF0/9jVgU7FwQd1JIbZbtOReystVYOy7eplCQo5Bztuuj7Foc%2bt/ChMuJw9FVesBPubpt2HbL1aUhW2zcpc1F7QK6dIYNIQl3NKmsbZqmTubSLXmcMr7zsSdLnNRXeXkgJ9COvbm1jZW0eC1zzPJMWTAh8vcxU2PgcjRfg4Mqo2ppz35n1qrNqlox0muzdPVTbQDafC27PIrw0xC6MejWlUY97rVWobrsnq3Lx8%2bSBZyl7MOgVa0JRiHDyvITKGVYcg6rCKvzt4MNfeq9sF&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20180808T132833Z&X-Amz-SignedHeaders=host&X-Amz-Expires=300&X-Amz-Credential=ASIAULPHH3IVDQYN7FMT/20180808/eu-west-1/s3/aws4_request&X-Amz-Signature=90912b8a47fd5bb373fcccff7c04b648b9c8572c72f1295e6d267964279a011d)

 2. Install Mysql and configure the agent:
 
    - install Mysql
```bash
sudo apt-get update 
sudo apt-get install mysql-server 
```
```bash
#firewall rule to allow mysql
sudo ufw allow mysql

#enabel mysql on system start
systemctl enable mysql

```
 
  - configure datadog agent integration
[More information about Mysql integration](https://docs.datadoghq.com/integrations/mysql/)
create database user for the Datadog Agent :

```bash
sudo mysql -e "CREATE USER 'datadog'@'localhost' IDENTIFIED BY '<datadog-dbuser-passowrd>';"

sudo mysql -e "GRANT REPLICATION CLIENT ON *.* TO 'datadog'@'localhost' WITH MAX_USER_CONNECTIONS 5;"

sudo mysql -e "GRANT PROCESS ON *.* TO 'datadog'@'localhost';"

sudo mysql -e "GRANT SELECT ON performance_schema.* TO 'datadog'@'localhost';"
```
check 
````bash
mysql -u datadog --password='<datadog-dbuser-passowrd>' -e "SELECT * FROM performance_schema.threads" && \
echo -e "\033[0;32mMySQL SELECT grant - OK\033[0m" || \
echo -e "\033[0;31mMissing SELECT grant\033[0m" 

mysql -u datadog --password='<datadog-dbuser-passowrd>' -e "SELECT * FROM INFORMATION_SCHEMA.PROCESSLIST" && \
echo -e "\033[0;32mMySQL PROCESS grant - OK\033[0m" || \
echo -e "\033[0;31mMissing PROCESS grant\033[0m"
````
Create or edit /etc/datadog-agent/conf.d/mysql.yaml file :

```yaml
init_config:

instances:
  - server: localhost
    user: datadog
    pass: <datadog-dbuser-passowrd>
    tags:
        - optional_tag1
        - optional_tag2
    options:
        replication: 0
        galera_cluster: 1
```
Restart the Agent  
```bash
sudo service datadog-agent restart
```





![enter image description here](https://s3.eu-west-1.amazonaws.com/marouane/datadog-screenshot/Screen%20Shot%202018-08-08%20at%2017.36.57.png?response-content-disposition=inline&X-Amz-Security-Token=AgoGb3JpZ2luEAkaDGV1LWNlbnRyYWwtMSKAAhS2jf2Dl7tHi78yWig86mSfeWXVrdzeCn%2bwvNssGSsFFuIDt65q0KPHVctdceb%2bgbgHvE0xLSg1XcPTteIqKdvdR8N6BEHrLPfZKFbCbrAlh3c8OSE8ri%2bBst5uhKidZbFlUBYJucwRZskm8m/5tvCNTZ36oev1fjjKXDuaXLFr6VTlZfpdGG6YPQiDkfSdfcOqJU7EzNKYBxdHqgw3kXjhTfDxnPYcAiM3FIK5MOxRF64%2brgZ3Fdy9CWJMagBtsNags4ojyphGW08eq0KEp8obYD0h/o8sPU/Bgdau2NI%2biQtScpLklldFobYMVucEpWm3AhUDGw0ocpKFdcTd/Jcq7AII7///////////ARABGgwyOTk1MjIwNTQ2OTgiDMXzAcwbjdjmowmYgSrAAhcbU3pmRtisjJmHevBe9j0S4CpxZOjYvoLz54G1tgcKab%2bRvh4AvpQH8gEjyuupeK65Y0vbdpPUPreD0IoCDrxKOabr2rxeFPCvpiUbqXEZ93QFYh/tp06Tf%2bfS26vVv65yCq1J6CDbI3fCzhJLR%2bC0fmZRtn2kaoF0/9jVgU7FwQd1JIbZbtOReystVYOy7eplCQo5Bztuuj7Foc%2bt/ChMuJw9FVesBPubpt2HbL1aUhW2zcpc1F7QK6dIYNIQl3NKmsbZqmTubSLXmcMr7zsSdLnNRXeXkgJ9COvbm1jZW0eC1zzPJMWTAh8vcxU2PgcjRfg4Mqo2ppz35n1qrNqlox0muzdPVTbQDafC27PIrw0xC6MejWlUY97rVWobrsnq3Lx8%2bSBZyl7MOgVa0JRiHDyvITKGVYcg6rCKvzt4MNfeq9sF&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20180808T153853Z&X-Amz-SignedHeaders=host&X-Amz-Expires=300&X-Amz-Credential=ASIAULPHH3IVDQYN7FMT/20180808/eu-west-1/s3/aws4_request&X-Amz-Signature=6e62a06a249181aef68fa088a15b2ab1eb431de580d7f9aa8b40a532aa886118)

# Monitoring Data

![enter image description here](/screenshots/monitorgraph.png)
![enter image description here](/screenshots/notificationtext.png)
![enter image description here](/screenshots/notificationtext.png)
![enter image description here](/screenshots/weeklydowntime.png)
![enter image description here](/screenshots/emailweekly.png)
![enter image description here](/screenshots/weekendowntime.png)
![enter image description here](/screenshots/emailweekend.png)



# Collecting APM Data

The APM Agent (also known as Trace Agent) is shipped by default with the Agent 6 in the Linux it will just need to enable it by editing the file `/etc/datadog-agent/datadog.yaml`:

```yaml
apm_config:
  enabled: true
  env: test  #trace tag
````
Installin datadog trace library:

```bash
pip install ddtrace
````
Edit the Flask app by adding: 

```python
from ddtrace import tracer
from ddtrace.contrib.flask import TraceMiddleware
```

And create the tracerobject:

```python
traced_app = TraceMiddleware(app, tracer, service="my-flask-app", distributed_tracing=False)
```
![enter image description here](/screenshots/FlaskAPM.png)
![enter image description here](/screenshots/Flaskapp.png)
![enter image description here](/screenshots/APMservice.png)


https://app.datadoghq.com/screen/409257/marouanes-screenboard-13-aug-2018-1623?page=0&is_auto=false&from_ts=1534166700000&to_ts=1534170300000&live=true

