![Virtualized Network Function](https://user-images.githubusercontent.com/4958202/136183752-19f32bb0-05e1-40b9-b92c-6b907fe6c418.png)

# teams_to_chime_gateway
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
(https://docs.microsoft.com/en-us/microsoftteams/business-voice/country-region-availability)\
4 - Amazon AWS Account

## AMI

The teams to chime gateway is an AMI available in the AWS marketplace. Search for the AMI number # (Waiting for AWS to publish the server)

## Instructions

Step #1 - Allocate an Elastic IP from Amazon AWS. 

Example: 44.1.1.1

Step #2 - In the DNS server associate an A record for the elastic IP previously allocated.

Example: sbcteams.wehostvoip.io

Step #3 - Add this domain to your office365 account. 

Verify the domain and add the appropriate record to your account

Step #4 - In the AWS Chime service create a voice connector and authorize outbound calls from the address of the teams to chime gateway

![image](https://user-images.githubusercontent.com/4958202/132652662-7776e6f8-7d23-473d-a187-8b6fb895ddf9.png)

Step #5 - Optionally you may create a phone number and direct incoming calls to the gateway 

Step #6 - Launch the AMI

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

Step #7 - Add the SBC to Microsoft Teams

In a microsoft windows notebook, in administrator mode run:

Example:

```
Install-Module -Name PowerShellGet -Force -AllowClobber
Install-Module -Name MicrosoftTeams -Force -AllowClobber
Update-Module MicrosoftTeams
Connect-MicrosoftTeams
New-CsOnlinePSTNGateway -Fqdn sbcteams.wehostvoip.io -SipSignalingPort 5061 -MaxConcurrentSessions 5 -Enabled $true -Bypass $None
```

Step #8 - Add a default PSTN usage

Example:

```
Set-CsOnlinePstnUsage -Identity Global -Usage @{Add="Default"}
```

Step #9 Create a voice route

Example:

```
New-CsOnlineVoiceRoute -Identity "Default" -NumberPattern "^\+" -OnlinePstnGatewayList sbcteams.wehostvoip.io -Priority 1 -OnlinePstnUsage Default
```

Step #10 Associate a user to a phone number

Example:

```
set-CsUser -Identity "flavio@vofficebr.onmicrosoft.com" -OnPremLineURI tel:+16692885660 -EnterpriseVoiceEnabled $true -HostedVoiceMail $true
```

Step #11 - Verify

1 - Verify if the gateway is up and running (Check Microsoft Teams, it may take up to 1 hour to synchronize)
2 - Test to see if you can make external calls, use the E164 +1... to make a call
3 - Test to see if you can receive external calls, call the AWS chime number and see if it rings. 

Step #12 - Troubleshooting. 

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
