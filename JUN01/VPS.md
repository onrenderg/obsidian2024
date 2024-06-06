6070932001267612
10/28 538

gstatparth@gmail.com
Aarth123!4


```txt
3 Month Free Trial

750 Hours per month

Try the $3.50/mo, $5/mo, or $10/mo plans for free for three months when using Linux/Unix

Try the $8, $12, or $20 plans for free for three months when using Windows

Try the $10 per node for free for three months when using Containers

Try the $15 database plans for free for three months when using Databases
```



https://aws.amazon.com/free/compute/lightsail/
https://lightsail.aws.amazon.com/ls/webapp/home/instances


https://ap-southeast-2.console.aws.amazon.com/console/home?region=ap-southeast-2#

https://www.fednetbank.com/corp/AuthenticationController?__START_TRAN_FLAG__=Y&FORMSGROUP_ID__=AuthenticationFG&__EVENT_ID__=LOAD&FG_BUTTONS__=LOAD&ACTION.LOAD=Y&AuthenticationFG.LOGIN_FLAG=1&BANK_ID=049&LANGUAGE_ID=001

https://docs.aws.amazon.com/en_us/lightsail/latest/userguide/amazon-lightsail-frequently-asked-questions-faq.html#lightsail-faq-data-transfer-charges



```bash
#Set the permissions to 600, which means only the owner can read and write the file.

chmod 600 ./awslighsail.pem


ssh -i ./awslighsail.pem ubuntu@13.126.121.104
```

```bash


sudo apt-get install sshuttle




sshuttle -r user@ssh_server_ip 0.0.0.0/0 -e 'ssh -i /path/to/private_key'

sshuttle -r ubuntu@13.126.121.104 0.0.0.0/0 -e 'ssh -i ./awslighsail.pem'

sshuttle --dns -r ubuntu@13.126.121.104 0.0.0.0/0 -e 'ssh -i ./awslighsail.pem'


```