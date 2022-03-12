---
title: "AWS Scaled Command Bash Script - Run AWS commands for many profiles"
date: 2022-03-12T10:42:14-08:00
draft: true
tags: [
        "pentest", "red", "cloud"
    ]

twitter:
  card: "summary_large_image"
  site: "@wunderwuzzi23"
  creator: "@wunderwuzzi23"
  title: "AWS Scaled Command Script"
  description:  "Running AWS CLI commands for many profiles"
  image: "https://embracethered.com/blog/images/2020/hacker.png"
---

One area that I have encountered quite often over the years is that during recon phase of a bug bounty hunt or pentest a set of AWS access keys are being discovered. 

Let's say you found 50 AWS access keys by drooling and [hunting](/blog/posts/2020/hunting-for-credentials/) through public Github repos and using other nifty tricks and means.

How do you go about checking their validity? And what do they have access to and provide the Bug Bounty Program or Blue Team the dates, times, and IP address when those keys were used? 

And, most importantly what if you want to run one command, but run them across all AWS profiles?

For that purpose, I have used a simple `bash` script called `aws-scaled-command.sh`:

```
#!/bin/bash

PROFILES=$(cat ~/.aws/credentials | grep  '\[*.\]' | sed 's/[][]//g')
CMD=$1

if [[ -v $1 ]]; then
  echo "Error: AWS command not provided: ./aws-scaled-command.sh 's3 ls'"
  exit
fi

echo "Using command: aws $1"
echo "Public Source IP Address:" $(curl --silent https://checkip.amazonaws.com)

for PROFILE in $PROFILES
do
    echo COMMAND START::$CMD::PROFILE::$PROFILE::DATE::$(date)
    echo -n "INFO:Getting caller identity: "
    
    IAM=""
    IAM=$(AWS_PROFILE=$PROFILE aws sts get-caller-identity | jq .Arn)
    
    if [[ $IAM != "" ]]; then
      
        echo $IAM
        echo "INFO::Running command aws $CMD: "
        
        AWS_PROFILE=$PROFILE aws $1
        
    fi

    echo COMMAND EXIT::$CMD::PROFILE::$PROFILE::USER::$IAM::EXIT CODE::$?
    
done

echo "Done."
```

## Usage

The following is a basic invocation to run `aws s3 ls` for all profiles inside `~/.aws/credentials`. Be careful to provide single quotes around the first argument passed into the script.

`./aws-scaled-command.sh 's3 ls'`

This will run the command against all profiles. Pipe the results to `output.txt` with something like `./aws-scaled-command.sh 's3 ls' > output.txt 2>&1` for parsing later.

Feel free to modify the output format. It's just a format (the weird double colons) I have been using for a longer time in my bash scripts. 

The output also contains the time and user that was used, as well as show the source IP on the top - so great for sharing with other stakeholders.

## Other Tools

There are of course many other tools, like [Weird AAL](https://github.com/carnal0wnage/weirdAAL) or some of the [Rhino Security tools, like Pacu](https://github.com/RhinoSecurityLabs/pacu) that allow with account enumeration and attacks. But for simple case I prefer the bash script. 

Worth highlighting here is also [`aws-vault`](https://github.com/99designs/aws-vault) which can help manage multiple keys.


## References

* [Rhino Security - AWS IAM PrivSec](https://github.com/RhinoSecurityLabs/AWS-IAM-Privilege-Escalation)
* [Rhino Security - Pacu](https://github.com/RhinoSecurityLabs/pacu)
* [Weird_AAL](https://github.com/carnal0wnage/weirdAAL)
* [AWS-Vault](https://github.com/99designs/aws-vault)
* [aws-scaled-command.sh](https://github.com/wunderwuzzi23/scratch/blob/master/aws/aws-scaled-command.sh)

