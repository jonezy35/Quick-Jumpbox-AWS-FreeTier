# Quick Jumpbox Using AWS Free Tier

If you have a homelab, you probably want to be able to access it when you're away from home. This is easy to accomplish using a VPN. However, if you want to access your homelab during your lunch break, or maybe you want to access your homelab during working hours (I do some development work on my home server that I plan to port over to some work projects when it's ready to be pushed to production) you probably need a jumpbox. You most likely can't VPN directly to your homelab from your work computer, so you have to use a jumpbox. 

A jumpbox is simply an intermediate server that we use to connect to our homelab. Our setup will look something like:
1. HTTPS connection to AWS
2. (This step is optional.. I'll explain below) SSH Connection from AWS CLI to [AWS Free Tier](https://aws.amazon.com/free/?all-free-tier.sort-by=item.additionalFields.SortRank&all-free-tier.sort-order=asc&awsf.Free%20Tier%20Types=*all&awsf.Free%20Tier%20Categories=*all) EC2 Instance (our jumpbox)
3. OpenVPN connection from EC2 Instance to homelab subnet
4. SSH Connection from EC2 Instance to Homelab VM through the OpenVPN Tunnel

There are certainly more advanced ways to set this up (utilizing reverse proxies) but this is a quick way to connect back to your homelab (for free for 12 months) without having to expose any ports on your home network (except for the port open to OpenVPN but this is very secure if setup correctly)

## Setting up OpenVPN
