![](https://user-images.githubusercontent.com/4958202/137627828-9a729407-f00e-4f55-99ab-c56911b4165a.png =250x250)

# Gateway from Microsoft Teams to AWS Chime Voice Connector
Microsoft Teams to AWS Chime Gateway 

If you need assistance to execute this procedures, please contact cloud@wehostvoip.io, assistance on configuration is cherged in an hourly basis. 

## Description

The teams to chime gateway is an AMI (Amazon Machine Image) dedicated to convert calls from MS teams using SIP and TLS to AWS Chime Voice Connector. This will allow MS Teams users to save on outbound calls avoiding the the cost of subscriber. We estimate a reduction from US$12.00 to US$2.00 per month per subcriber in telephpny costs, by using the gateway

** Please consult, if you want to install this gateway in your internal network or if you want an image for your own ITSP provider **
(cloud_at_wehostvoip.io)

## Prerequisites

To use the teams to chime gateway you should have:

1 - Microsoft 365 account\
2 - Microsoft Teams, any business license.\
3 - Microsoft Phone System Addon license or “Business Voice without calling plan” Add On License\
(https://docs.microsoft.com/en-us/microsoftteams/business-voice/country-region-availability) \
4 - Amazon AWS Account\
5 - A Tecnhnician with basic knowledge on TCP/IP, Domain Name System, MS Teams, Office 365, MS PowerShell and AWS Console

## AMI

The teams to chime gateway is an AMI available in the AWS marketplace. Search for the AMI number # (Waiting for AWS to publish the server)

## Instructions

### Part 1 - AWS Chime and Session Border Controller

These tasks are made in the AWS Console, you will need to have appropriate rights to administer your AWS account


Step #1 - In the AWS Chime service create a voice connector and authorize outbound calls from the address of the teams to chime gateway. Use UDP and a name and password

![image](https://user-images.githubusercontent.com/4958202/132652662-7776e6f8-7d23-473d-a187-8b6fb895ddf9.png)

Step #2 - Optionally you may create a phone number and direct incoming calls to the gateway 

Step #3 - In the AWS console with an account with appropriate rights, allocate an Elastic IP from Amazon AWS. 

Example: 44.1.1.1

Step #4 - In the DNS server associate an A record for the elastic IP previously allocated. You will have to access your DNS provider (e.g.GoDaddy, Cloudflare, Route53) and create an A record for your SBC. 

Example: sbcteams.wehostvoip.io

Step #5 Instantiate the Session Border Controller launching the appropriate AMI from the MarketPlace

To configure the AMI, check the example of user data. You can launch the instance with the recommended t2.medium type

** Very Important **

In the AWS EC2 instance details, Fill the field <b>user-data</b> with the configuration, this is the standard way to provision the AMI.

![image](https://user-images.githubusercontent.com/4958202/132643831-001d6dda-7ff3-4e44-8cea-a1ae98cfd43d.png)

In the user data, provide your teams gateway domain name and the AWS VP_HOST, all other fields are optional. Never open the port 5060 to the world and allow calls only from the gateway. The security group provided allow only calls from MS Teams to AWS Chime. 

```
#!/bin/bash
export SBC_FQDN='sbcteams.wehostvoip.io'
export VP_HOST='fact1wkfr4ut1hlmf.voiceconnector.chime.aws'
export VP_PORT='5060'
export VP_USERNAME=''
export VP_PASSWORD=''
export VP_PROTOCOL='UDP'
cd /usr/src/fsteams/config
./install.sh sbcteams.wehostvoip.io 
```

Step #6 - Create a security group for your SBC

I tried to make this automatic on the instance creation. Unfortunately, AWS Marketplace does not support ranges in the SG creation. 

Add a security group with the following rules:

Custom UDP UDP 1024 - 65535 34.212.95.128/25 AWS Chime Media
HTTP TCP 80 0.0.0.0/0 Lets Encrypt
Custom TCP TCP 8000 0.0.0.0/0 Console
Custom UDP UDP 1024 - 65535 99.77.253.0/24 AWS Chime Media
Custom TCP TCP 5060 - 5061 99.77.253.0/24 AWS Chime Signaling
Custom UDP UDP 1024 - 65535 34.223.21.0/25 AWS Chime Media
Custom UDP UDP 5060 99.77.253.0/24 AWS Chime Signaling
Custom TCP TCP 5061 52.114.14.70/32 MS Teams Signaling
Custom TCP TCP 5061 52.114.7.24/32 MS Teams Signaling
Custom UDP UDP 16384 - 32768 52.112.0.0/14 MS teams Media
Custom UDP UDP 1024 - 65535 52.55.62.128/25 AWS Media
Custom TCP TCP 5061 52.114.75.24/32 MS Teams Signaling
Custom TCP TCP 5060 - 5061 3.80.16.0/23 AWS Chime Range
Custom UDP UDP 5000 - 65000 3.80.16.0/23 AWS Chime Range
Custom TCP TCP 5061 52.114.76.76/32 MS Teams Signaling
Custom TCP TCP 16384 - 32768 52.120.0.0/14 MS teams Media
Custom TCP TCP 5061 52.114.148.0/24 MS Teams Signaling
Custom TCP TCP 5061 52.114.14.46/32 MS Teams Signaling
Custom UDP UDP 1024 - 65535 52.55.63.0/25 AWS Chime Media

### Part 2 - Microsoft Teams Tenant Configuration

Step #1 - Add your domain to your office365 account. 

This is probably the most difficult step in the configuration. You need to be administrator of your Office 365 system to perform this task. You have to verify the domain and add the skype for business records in the system. You have to perform three steps

1 - Add the domain to your Office365 tenant\
2 - Verify the domain\
3 - Add the specific records for skype for business

For the SBC domain you don't need to enable the records for e-mail.

Step #2 - Add the SBC to Microsoft Teams

Use an windows notebook with Windows 10 for this task. 

Start Powershell in administrator mode and run: 

Example:

```
Install-Module -Name PowerShellGet -Force -AllowClobber
Install-Module -Name MicrosoftTeams -Force -AllowClobber
Update-Module MicrosoftTeams
Connect-MicrosoftTeams
New-CsOnlinePSTNGateway -Fqdn sbcteams.wehostvoip.io -SipSignalingPort 5061 -MaxConcurrentSessions 5 -Enabled $true -Bypass $None
```

Step #3 - Add a default PSTN usage

Example:

```
Set-CsOnlinePstnUsage -Identity Global -Usage @{Add="StandardUsage"}
```

Step #4 Create a voice route

Example:

```
New-CsOnlineVoiceRoute -Identity "SimpleVoiceRoute" -NumberPattern "^\+" -OnlinePstnGatewayList sbcteams.wehostvoip.io -Priority 1 -OnlinePstnUsage StandardUsage
```

Step #5 Associate a user to a phone number

Example:

```
set-CsUser -Identity "flavio@vofficebr.onmicrosoft.com" -OnPremLineURI tel:+16692885660 -EnterpriseVoiceEnabled $true -HostedVoiceMail $true
```

Step #6 Create a voice routing policy called Standard Policy

```
New-CsOnlineVoiceRoutingPolicy -Identity "StandardPolicy" -OnlinePstnUsages "StandardUsage"
```

Step #7 Associate a policy to the username

```
Grant-CsOnlineVoiceRoutingPolicy -Identity "flavio@vofficebr.onmicrosoft.com" -PolicyName "StandardPolicy"
```

Step #8 - Verify

1 - Verify if the gateway is up and running (Check Microsoft Teams, it may take up to 1 hour to synchronize)
2 - Test to see if you can make external calls, use the E164 +1... to make a call
3 - Test to see if you can receive external calls, call the AWS chime number and see if it rings. 

Step #9 - Troubleshooting. 

Connect the servert thru ssh (putty, ssh)
Use sudo su to get root access

There are a few tools to troubleshoot the installation:

```
teams1_status - Show the status of the first teams gateway
teams2_status - Show the status of the first teams gateway
teams3_status - Show the status of the first teams gateway
chime_status - Show the status of the chime trunk
```

```
=================================================================================================
Name            ms1
Profile         default-teams
Scheme          Digest
Realm           sip.pstnhub.microsoft.com
Username        default
Password        yes
From            <sip:default@sbcteams.wehostvoip.io>
Contact         <sip:default@sbcteams.wehostvoip.io:5061;transport=tls;gw=ms1>
Exten           default
To              sip:default@sip.pstnhub.microsoft.com
Proxy           sip:sip.pstnhub.microsoft.com
Context         default-TEAMS
Expires         3600
Freq            3600
Ping            1631176168
PingFreq        25
PingTime        14.58
PingState       1/1/1
State           NOREG
Status          UP
Uptime          492s
CallsIN         0
CallsOUT        0
FailedCallsIN   0
FailedCallsOUT  0
=================================================================================================
``` 

To capture packets from the server use:

```
start_capture
sngrep 
```

If you want to see the console

```
console
```

To see the general status of the server use

```
status
```
