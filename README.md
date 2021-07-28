
1. SSH into proxmox node and become root user

```bash
apt update
apt install -y libsasl2-modules
```

2. Enable 2FA for the gmail account that will be used

3. Create app password for the account.
  a. Go to [App Passwords](https://security.google.com/settings/security/apppasswords) and generate a new password for Proxmox scope
  b. Select app: Mail
  c. Select device: Other
  d. Type in: Proxmox
  
4. Write gmail credentials to file. Again, make sure you are root.

```bash
echo "smtp.gmail.com youremail@gmail.com:yourpassword" > /etc/postfix/sasl_passwd
postmap hash:/etc/postfix/sasl_passwd
chmod 600 /etc/postfix/sasl_passwd
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

**IMPORTANT**: Comment out the existing `relayhost=` line above what we pasted in, since that would cause a duplicate key to be present.

7. Reload postfix
```bash
postfix reload
```

8. Test to make sure everything is hunky-dory.
```bash
echo "sample message" | mail -s "sample subject" anotheremail@gmail.com
```
