# Quick Jumpbox Using AWS Free Tier

If you have a homelab, you probably want to be able to access it when you're away from home. This is easy to accomplish using a VPN. However, if you want to access your homelab during your lunch break, or maybe you want to access your homelab during working hours (I do some development work on my home server that I plan to port over to some work projects when it's ready to be pushed to production) you probably need a jumpbox. You most likely can't VPN directly to your homelab from your work computer, so you have to use a jumpbox. 

A jumpbox is simply an intermediate server that we use to connect to our homelab. Our setup will look something like:
1. HTTPS connection to AWS
2. (This step is optional.. I'll explain below) SSH Connection from AWS CLI to [AWS Free Tier](https://aws.amazon.com/free/?all-free-tier.sort-by=item.additionalFields.SortRank&all-free-tier.sort-order=asc&awsf.Free%20Tier%20Types=*all&awsf.Free%20Tier%20Categories=*all) EC2 Instance (our jumpbox)
3. OpenVPN connection from EC2 Instance to homelab subnet
4. SSH Connection from EC2 Instance to Homelab VM through the OpenVPN Tunnel

There are certainly more advanced ways to set this up (utilizing reverse proxies) but this is a quick way to connect back to your homelab (for free for 12 months) without having to expose any ports on your home network (except for the port open to OpenVPN but this is very secure if setup correctly)


## Prerequisites
This post assumes that you are either running PfSense OR you know how to set up a VPN connection on you respective router. I have PfSense setup on dedicated hardware but [here](https://docs.netgate.com/pfsense/en/latest/recipes/virtualize-esxi.html) is Netgate's documentation on virtualizing PfSense. 


## Setting up OpenVPN
Instead of rehashing all of the stuff that's been put together already, I'll link you to a great [youtube video](https://www.youtube.com/embed/PgielyUFGeQ?autoplay=0&cc_lang_pref=en&cc_load_policy=0&color=0&controls=1&fs=1&h1=en&loop=0&rel=0) that goes over setting up OpenVPN on PfSense.

  ### One Note: I have my homelab on a separate VLAN from the rest of my network. I recommend this because you can set the VPN up to only allow connections to the Homelab VLAN and NOT to the rest of your network. 

## Setting up the EC2 Instance

These steps should be done at home, when you're not on your home network. 

1. You first need to navigate to the [AWS management console](aws.amazon.com). Create an account and then you need to launch a t2.micro instance. For this walkthrough we are using an Amazon Linux AMI. 
2. Once the t2.micro instance is spun up, you need to SSH to the instance from the same computer that you downloaded the openvpn file to when you setup OpenVPN and exported the file. 
3. scp the file over to your EC2 Instance.

## Connecting to your homelab

You should now be able to login to your EC2 instance (via SSH or via EC2 Instance Connect) and run `openvpn your_vpn_file` and it will establish a VPN connection back to your homelab. You will have to provide your username and password. If you want to run this in the background, then you need to create a credential file that contains your username and password. For examle

```
username
password
```
Then you execute the openvpn file by typing `sudo openvpn --config your_vpn_file --auth-user-pass your_credential_file &` The `&` runs it in the background so you can still use your terminal.

If you get the error: "Unrecognized option or missing or extra parameter(s) in ...  data-ciphers" you need to go into your ovpn file and delete the two lines `data-ciphers` and `data-ciphers-fallback`

## If you can't access EC2 via SSH or via EC2 Instance Connect

If your work proxy doesn't allow you to use EC2 Instance Connect (as mine doesn't) and you aren't able to connect via ssh, how are you supposed to connect to your EC2 instance? There is a workaround. You can access AWS CloudShell and ssh to the EC2 instance from cloudshell.

1. First, you need to create an ssh key pair. `ssh-keygen -t ed25519`
2. Now, you need to send the public key over to your EC2 instance `aws ec2-instance-connect send-ssh-public-key --instance-id your_instance_id --instance-os-user ec2-user --ssh-public-key file://your_key.pub`
3. SSH to your EC2 instance
    - Open a new tab and navigate to your instance
    - Click connect and navigate to the ssh client tab
    - Copu the ec2-user@ address.
    - Run `ssh -o "IdentitesOnly=yes" -i your_key ec2-user@your_instance_address`
4. You should now be ssh'd into your EC2 instance and can execute your OpenVPN file.

## After the VPN connection is established
Once you have the VPN connection established, you should be able to SSH to your homelab VM's as normal. 
