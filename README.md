# securethebox-challenge

# Tips
- User must know some basics of using Linux OS
- bash/sh, vi, ps, netstat
- Basic knowledge of what a WAF does
- What are regular expressions
- SQL syntax
- OWASP top 10

# Requirements for Local Development
- Edit your local dns configuration file (/etc/hosts for mac)
```
127.0.0.1       monitor.securethebox.us
127.0.0.1       splunk.securethebox.us
127.0.0.1       waf.securethebox.us
127.0.0.1       waf-editor.securethebox.us
127.0.0.1       app.securethebox.us
127.0.0.1       editor-app.securethebox.us
```
- Install npm
- Install docker
- Install required packages:
```
cd securethebox-challenge
npm install
```
- Run Challenge:
```
npm run challenge
```
- View Challange:
```
visit waf.securethebox.us to view the App behind ModSecurity
visit waf-editor.securethebox.us to use CloudCmd to edit the files of modsecurity
visit app.securethebox.us to view the app without the firewall
visit editor-app.securethebox.us to use CloudCmd to edit the files of vulnerable app
```

# Notes
- Instructions for AWS ECS coming soon...