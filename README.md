# Pushing a website to DigitalOcean via nginx on linux

1 First of all, go to Digital Ocean or any cloud provider and go to DNS routing.

2 Create new record, set the site name and assign the ip address of the computer where the nginx works Note: After finishing step 2, it will take some time to change global dns settings, during that time do the other steps

3 Place your website inside the server via cloning github/gitlab repo or use scp to push folder to the server. Tip: Dont place your folder inside username.

> `sudo -u www-data stat /your/route/here`  
> Note: Also check whether the folder is accessible via internet using the command. If it shows status of two or three files. then it is ok.

4 Now, move to /etc/nginx/ folder where the internal routing is handled, move to sites-available folder where the domains and sub-domains are handled and create a file using any text-editor you prefer.

5 eg: inside /etc/nginx/sites-available type vi your-file-name inside theeditor press "i" key first and write the following (note:This is for simple static website further documentation will be found on nginx website).

```nginx
server {
listen 80;
// The following three lines are used to debug. These are not necessary.
charset UTF-8;
access_log  /root/cbme-website/personalsite.access.log;
error_log /root/cbme-website/personalsite.error.log;
server_name your-website;
 root /your/directory/here;
 index index.html;
 // This is a comment here: the below lines are the actual processing of the uri given,
 // There can be multiple request handling,more info can be found on the website.
 location / { // This is for static files
   try_files $uri /index.html;
  }
 // The below is for a nodejs project running on port
  location / {
    proxy_pass http://url-given-in-project
  }
}
```

6 The sites-available folder contains documents which gives list of websites present, this doesnt complete our work. we need to push a soft link to sites-enabled folder present in /etc/nginx/. The sites-enabled folder works just as the name suggests. To do a soft link, type the following

> `ln -s /etc/nginx/sites-available/your-file-name /etc/nginx/sites-enabled/`

7 Check whether that you have performed step 5 correctly using:

> `nginx -t`  
>
>if this shows ok, then restart the nginx service via:  
>
> `sudo systemctl restart nginx`
>  
>A shorter way to do step 7 is  
>
> `nginx -t && nginx -s reload`

8 During the steps 3 to 5, your DNS should have been set. Now your website only receives HTTP request. To make the website HTTPS, we need to receive certificate via certbot.

>First, download the Let’s Encrypt client >certbot by writing these commands one by one:
>
> ```bash
> apt-get update
> sudo apt-get install certbot
> apt-get install python-certbot-nginx
> ```
>
>get certificate by:  
> `sudo certbot --nginx -d your-website`
>If it gets successful, you get the message starting like this:
>
>```bash
>Congratulations! You have successfully >enabled your-website...
>```
>

This step changes the file created at "Step 4" a bit and enabled https access. Note that certificate is valid only for 90 days. To renew the certificate do this.

```bash
crontab -e //opens the file
write the following inside the file.
0 12 \* \* \* /usr/bin/certbot renew --quiet
```

Now, your website may be accessible
