---
layout: post
title:  "How to backup an iOS device on Linux (and check it for compromise)"
lang: en
tags: [en, ios, backup, forensics]
category: tutorial
published: true
---

After I moved from MacOS to Ubuntu as my daily driver I still kept a Windows VM to run some software, and I started using iTunes to back up my phone in that VM. And only when I began to get interested in iOS forensics I discovered, and it came to me as a surprise, that it is actually possible to back up (and restore) an iOS device on Linux natively using software toolkit called `libimobiledevice-utils`. Originally doing (and then decrypting) a backup was just a step to examine a device for a potential compromise. 

## backup

So on Ubuntu in order to make a backup I install the toolkit:

```shell
sudo apt install libimobiledevice-utils
```

Then I connect the phone and check if the software can see it:

```shell
ideviceinfo
```

It should output the device's information. Then try to enable encryption to confirm that it is enabled. All of the backups must be encrypted not just for security reasons but also because encrypted backups contain more information:

```shell
idevicebackup2 -i encryption on
```

Then do the backup:

```shell
idevicebackup2 backup --full /mnt/backups/phone-backup
```

To restore the backup (haven't tried it yet) I would erase the device first and then do:

```shell
idevicebackup2 restore --system --settings /mnt/backups/phone-backup
```

It might be useful in some cases (not just when doing forensics) to decrypt an encrypted backup to save the files from it if no device available to roll out the backup to. For that we need the MVT.

Build the `mvt` docker image:

```shell
git clone https://github.com/mvt-project/mvt.git
cd mvt
docker build -t mvt .
```


Run the extraction:

```shell
 MVT_IOS_BACKUP_PASSWORD="mypassword" docker run -it \
  --user "$(id -u):$(id -g)" \
  -e MVT_IOS_BACKUP_PASSWORD \
  -v /mnt/encrypted_storage:/mnt/encrypted_storage:rw \
  -v /mnt/backups:/mnt/backups:ro \
  mvt \
  mvt-ios decrypt-backup -d /mnt/encrypted_storage/phone-decrypted /mnt/backups/phone-backup
```

note the space in the beginning of the command

## check for compromise

Once the backup is decrypted, it is possible to run a check for IOCs:

`mkdir iocs` and then download IOCs:

```shell
docker run -it \
  --user "$(id -u):$(id -g)" \
  -v /mnt/encrypted_storage:/mnt/encrypted_storage:rw \
  -v "$PWD/iocs:/home/ubuntu/.local/share/mvt/indicators:rw" \
  mvt \
  mvt-ios download-iocs
```

`mkdir reports` and the check the backup for IOCs:

```shell
docker run -it \
  --user "$(id -u):$(id -g)" \
  -v /mnt/encrypted_storage:/mnt/encrypted_storage:rw \
  -v "$PWD/iocs:/home/ubuntu/.local/share/mvt/indicators:rw" \
  -v "$PWD/reports:/reports:rw" \
  mvt \
  mvt-ios check-backup --output /reports/ /mnt/encrypted_storage/phone-decrypted/
```

`reports/` **will contain sensitive information** and must be stored accordingly.

## references

- https://docs.mvt.re/en/latest/ios/methodology/