# linux-server-configuration

### Steps to set up linux server and configure it

##### A. Sign up for [Amazon Lightsail](https://lightsail.aws.amazon.com) account.
##### B. Create a new [Amazon Lightsail instance](https://lightsail.aws.amazon.com/ls/webapp/create/instance).
Select OS only - choose Ubuntu 16.04 LTS
##### C. Adding TCP 2200 port in the firewall
1. Go to the instance
2. Click on Networking tab
3. Add a new custom firewall with Application -> Custom, Protocol -> TCP, and Port range -> 2200
##### D. Downloading SSH key
 1. [Download default SSH key pair](https://lightsail.aws.amazon.com/ls/webapp/account/keys)
 2. Store the key under ~/.ssh folder in your **LOCAL** machine.
 

