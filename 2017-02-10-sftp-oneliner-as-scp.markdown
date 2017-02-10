Title: SFTP oneliner (as SCP)
Date: 2017-02-10 12:12:17 +0200
Slug: 2017-02-10-sftp-oneliner-as-scp
category: posts
tag: linux, notetoself

Quick note to self:

[Secure Copy (scp)](https://linux.die.net/man/1/scp) (a one line command very easy to include in bash scripts) can be replaced by a similar [Secure File Transfer (sftp)](https://linux.die.net/man/1/sftp) command, but the documentation is not very clear...

For instance, this command to transfer a file to a remote host using SCP:

``` scp src_file user@remote_host:/remote/path/dst_file ```

Can be replaced by this one, using SFTP:

``` sftp user@remote_host:/remote/path/ <<< $'put  src_file dst_file' ```

Example:

```
# sftp root@192.168.1.100:/tmp/ <<< $'put /tmp/test test_sftp'
Connected to 192.168.1.100.
Changing to: /tmp/
sftp> put /tmp/test test_sftp
Uploading /tmp/test to /tmp/test_sftp
/tmp/test                                          100%    0     0.0KB/s   00:00
#
```

The idea is taken from a few StackOverflow threads, all the credit to them... I decided to put it together here, so I can find it easily.

Later
