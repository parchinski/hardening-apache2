---
layout: default
---

# Hardening Apache2 on Debian Bookworm

## Common Apache Web Server Attacks

I reasearched the most common attacks on Apache2, and one I came across and wanted to address are DDos attacks. DDos attacks are a type of attack that attempts to overwhelm the web server by keeping as many connections open as possible for as long as possible. This can be done by sending a large amount of requests to the web server or by sending a small amount of requests but keeping the connection open for a long time. I will address this attack in my best practices section.

## Key Components and Best Practices for Apache Security on Debian 12 in VirtualBox

- **Regular Vulnerability Assessment**: Update Apache against known vulnerabilities and perform regular security testing.
- **Adopt the Principle of Least Privilege**: Disable unnecessary services and run Apache with minimal permissions.
- **Prevent DDoS Attacks**: Utilize Apache modules like mod_reqtimeout and other techniques to mitigate Slowloris attacks.

## Best Practices for Apache2 on Debian 12 Based on Research

1. **Disable Unnecessary Modules**: I disabled unnecessary modules to reduce the attack surface of the Apache web server. To begin this process I deleted all running modules from Apache2 then I added only the essentials need for the web server to function. These essentials were mpm_prefork for the Apache2 web server to function, dir and mime for the web server to serve files, autoindex for the web server to display a list of files in a directory if no index file is present, and log_config for the web server to log activity. and authz_core for the webserver to authenticate users. I will decide later if I want to add more modules to the web server but only for additional encryption/security.

2. **Enable reqtimeout to Protect against DDos Attacks**: I enabled the reqtimeout module to protect against Slowloris attacks. Slowloris attacks are a type of DDoS attack that attempts to overwhelm the web server by keeping as many connections open as possible for as long as possible. The reqtimeout module allows the web server to timeout connections that are not sending requests in a timely manner. I did this by adding the following code to the apache2.conf file.
   This should limit the amount of time a connection can be open to 20 seconds and the amount of time a request can be sent to 40 seconds. This should be enough time for a user to send a request and receive a responses but not enough time for a Slowloris attack to overwhelm the web server.

```
<IfModule reqtimeout_module>
    RequestReadTimeout header=20-40,MinRate=500 body=20,MinRate=500
</IfModule>
```

5. **Ensure That Apache2 and Debian 12 are Regularly Updated**: I update by running the following commands.

```
sudo apt update
sudo apt upgrade
```

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
