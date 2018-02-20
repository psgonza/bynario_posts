Title: PSA: MiniKeePass and key files
Date: 2018-02-17 00:01:17 +0200
Slug: 2018-02-17-psa_minikeepass_and_key_files
category: posts
tags: keepass

In case you plan to use MiniKeePass on your iphone with key files (which I guess is highly recommended if you plan to share your kdb via Dropbox/Drive/you_name_it), pay special attention to the last line of their help page:

```
Using Key Files

MiniKeePass can open KeePass 1.x/2.x files that use a key file instead of or in <br>
addition to a password.

Steps:

Load your KeePass database and key file in MiniKeePass using iTunes, Dropbox, etc.
Open your KeePass file in MiniKeePass
When prompted for your password you can enter a password and/or select a key file

Note: MiniKeePass will automatically select your key file if it has the same <br>
filename as your KeePass file but with a .key extension.
```

Basically, you have to upload your data base and the key file ** with the same name **(ie: MyDB.kdb and MyDB.key) to your_cloud_storage_service_here, and then, open both of them with MiniKeePass. Otherwise, all you will see in the Key-file screen is "None". 

I learnt it the hard way and I lost 15 minutes of my life trying to import a key file with a different name than the KeePass database.

The devil is in the detail, as usual.... ;)

///psgonza

[KeePass](https://keepass.info/) is great, you should totally use. [MiniKeePass](https://minikeepass.github.io/) is just fine, does the job too.
