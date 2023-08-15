# Apache HTTP Server 2.0 vulnerabilities notes

## Common Apache Web Server Attacks

- **Distributed Denial of Server (DDoS) Attack**: A DDoS attack on an Apache webserver attempts to overwhelm it by crowding it with several thousand HTTP requests at the same time, this is beyond Apache's load limits. as a result, the system will slow and perhaps stop functioning completely.
- **Buffer Overflow**: A buffer overflow attack takes advantage of a condition created by a poorly written code to overrun its capacity or the buffer's boundary and overwrite adjacent memory locations. This can cause the system to crash and allow the attacker to gain control of the system.
- **Root Exploit**: This attack method enables a malicious hacker to gain root access through the applications running as a root process on the Apache server. This allows the attacker to gain full control of the system.
- **Website Defacement**: This allows an attacker to inject malicious code into a hosted application to gain access to resources or cause harm when the website is requested it will display defaced web pages.
- **Directory Traversal**: This attack allows an attacker to access beyond the Apache web root directory restricted directories and execute commands outside of the web server's root directory.
- **Misconfiguration Attacks**: If default Apache configuration files are used, or unnecessary services are enabled, attackers can compromise the Apache webserver through various attacks such as password cracking, injection attacks, and others.

## Key Components and Best Practices for Apache Security on Debian 12 in VirtualBox

- **Access Control**: Implement strong authentication and authorization, including multi-factor authentication (MFA).
- **Confidentiality and Integrity**: Utilize encryption, SSL/TLS, and digital certificates to ensure data confidentiality and integrity.
- **Availability**: Design the Apache server for high availability, fault tolerance, and resilience against DDoS attacks.
- **OS Hardening**: Patch and secure the underlying OS, minimize exposure to threats, and follow Apache Server Announcements.
- **Regular Vulnerability Assessment**: Update Apache against known vulnerabilities and perform regular security testing.
- **Adopt the Principle of Least Privilege**: Disable unnecessary services and run Apache with minimal permissions.
- **Implement Access Control Measures**: Utilize password-based credentials, IP whitelisting, and disable .htaccess files.
- **Prevent DDoS Attacks**: Utilize Apache modules like mod_reqtimeout and other techniques to mitigate Slowloris attacks.
- **Install a Web Application Firewall (WAF)**: Use ModSecurity to monitor, filter, and block HTTP traffic.
- **Monitor and Log Relevant Data**: Enable Apache logging modules and utilize tools like Fail2Ban for monitoring and logging.

## Best Practices for Apache2 on Debian 12 Based on Research

1. **Added additional Network Adapter to VirtualBox Network Configuration**: I added a NAT adapter and a Host-only adapter to my VirtualBox network configuration. The NAT adapter allows the VM to connect to the internet in order to ensure latest versions of the OS and of Apache2 and the Host-only adapter allows the VM to connect to the host machine.

2. **Disable Unnecessary Modules**: I disabled unnecessary modules to reduce the attack surface of the Apache web server. To begin this process I deleted all running modules from Apache2 then I added only the essentials need for the web server to function. These essentials were mpm_prefork for the Apache2 web server to function, dir and mime for the web server to serve files, autoindex for the web server to display a list of files in a directory if no index file is present, and log_config for the web server to log activity. and authz_core for the webserver to authenticate users. I will decide later if I want to add more modules to the web server but only for additional encryption/security.

### Commands to Enable and Disable modules

```
sudo a2dismod <module_name>
sudo a2enmod <module_name>
```

3. **Enable reqtimeout to Protect against DDos Attacks**: I enabled the reqtimeout module to protect against Slowloris attacks. Slowloris attacks are a type of DDoS attack that attempts to overwhelm the web server by keeping as many connections open as possible for as long as possible. The reqtimeout module allows the web server to timeout connections that are not sending requests in a timely manner. I did this by adding the following code to the apache2.conf file.
   This should limit the amount of time a connection can be open to 20 seconds and the amount of time a request can be sent to 40 seconds. This should be enough time for a user to send a request and receive a responses but not enough time for a Slowloris attack to overwhelm the web server.

```
<IfModule reqtimeout_module>
    RequestReadTimeout header=20-40,MinRate=500 body=20,MinRate=500
</IfModule>
```

4. **Enable SSL Encryption**: the process is complex and requires an ssl provider here is a good tutorial for Apache and Debian I will maybe implement this later if bored: [SSL Tutorial] (https://www.ssltrust.com.au/help/setup-guides/debian-ubuntu-ssl-install-guide)

5. **Ensure That Apache2 and Debian 12 are Regularly Updated**: I update by running the following commands.

```
sudo apt update
sudo apt upgrade
```

6. **Enable Fail2Ban to automatically block IP addresses that show malicious signs such as too many password failures**

7. **Enable ModSecurity to monitor, filter, and block HTTP traffic**

8. **Enable Apache Logging Modules**: I would enable the logging modules to log activity on the web server. This will work after I download the modules: log_config, logio, and mod_log_forensic. Then I would add the following code to the apache2.conf file.

```
# Log format
LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-agent}i\"" combined
LogFormat "%h %l %u %t \"%r\" %>s %b" common
LogFormat "%{Referer}i -> %U" referer
LogFormat "%{User-agent}i" agent

# Access log
CustomLog /var/log/apache2/access.log combined
```

9. **Enable Apache2 to run with minimal permissions**: Make sure www directory at /var/www is owned by www-data user and group. This is the user and group that Apache2 runs under. This will ensure that the www folder has the lowest possible permissions. I did this by running the following commands.

```
sudo chown -R www-data:www-data .
```

10. **Disable .htaccess files and Diabled Directory Browsing**: I disabled .htaccess files because they are not needed and can be used to exploit the web server. I disabled directory browsing because it is not needed and can be used to exploit the web server. I did this by adding the following code to the apache2.conf file.

```
<Directory /var/www/>
    AllowOverride None
    Require all granted
    Options -Indexes
</Directory>
```

### Conclusion and My Implementations

Apache2 can be difficult to make safe especially at scale but once I learn encryption better I think I will be mroe confident with it. I have also attached some images showing my implementations. of two of the best practices I implemented. The first shows how the owner and group of the www folder and all of its contents are owned by www-data. The second shows how I successfully minimized the amount of modules running on the Apache2 web server to only what was necessary. After my testing the web server was still able to serve files.

## Sources

- [Apache Web Server Security](https://www.comparitech.com/net-admin/apache-web-server-security/)
- [Apache Docs Vulnerabilities](https://httpd.apache.org/security/vulnerabilities_20.html)
