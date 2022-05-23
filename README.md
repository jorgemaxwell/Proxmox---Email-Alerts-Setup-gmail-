# Proxmox - SMTP Setup (gmail)

1. SSH into proxmox node and become root user. Run the following commands to download extra software dependencies we'll need.

    ```bash
    apt update
    apt install -y libsasl2-modules
    ```

2. Enable 2FA for the gmail account that will be used by going to [security settings](https://myaccount.google.com/security)

3. Create app password for the account.
    1. Go to [App Passwords](https://security.google.com/settings/security/apppasswords)
    2. Select app: Mail
    3. Select device: Other
    4. Type in: Proxmox
  
4. Write gmail credentials to file and hash it. Again, make sure you are root.

    ```bash
    echo "smtp.gmail.com youremail@gmail.com:yourpassword" > /etc/postfix/sasl_passwd
    
    # chmod u=rw
    chmod 600 /etc/postfix/sasl_passwd
    
    # generate /etc/postfix/sasl_passwd.db
    postmap hash:/etc/postfix/sasl_passwd
    ```


5. Open the Postfix configuration file with editor of your choice.

    ```bash
    nano /etc/postfix/main.cf
    ```

6. Append the folling to the end of the file:
    ```text
    relayhost = smtp.gmail.com:587
    smtp_use_tls = yes
    smtp_sasl_auth_enable = yes
    smtp_sasl_security_options =
    smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
    smtp_tls_CAfile = /etc/ssl/certs/Entrust_Root_Certification_Authority.pem
    smtp_tls_session_cache_database = btree:/var/lib/postfix/smtp_tls_session_cache
    smtp_tls_session_cache_timeout = 3600s
    ```

    **IMPORTANT**: Comment out the existing line containing just `relayhost=` since we are using this key in our configuration we just pasted in.
    
    <img width="741" alt="Screen Shot 2022-05-23 at 12 04 15 AM" src="https://user-images.githubusercontent.com/12147036/169765896-ebacf0af-f819-4560-b6ed-3dbc227e6766.png">

7. Reload postfix
    ```bash
    postfix reload
    ```

8. Test to make sure everything is hunky-dory.
    ```bash
    echo "sample message" | mail -s "sample subject" anotheremail@gmail.com
    ```

