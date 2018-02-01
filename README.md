# r53
AWS Route 53 DNS Record Updater

## Prerequisite
* awscli
* jq
* a hosted zone which has already been created.

## Install
Clone this repo in a directory of your choice. Optionally, add the cloned directory to your $PATH.
```
$ cd ~/app
$ git clone git@github.com:genneko/r53.git
or
$ git clone https://github.com/genneko/r53.git
```

## Initial Setup
Run 'r53 configure' and input your IAM account and a hosted zone information. They will be written in your $HOME/.r53rc file.
```
$ ./r53 configure
AWS CLI Profile [default]: dns
Hosted Zone Domain []: example.jp
Hosted Zone ID [XXXXXXXXXXXXXX]:
```

## Usage
```
    r53 [opts] update [-t <type>] [-T <ttl>] [-C <comment>] <name> <value>
    r53 [opts] delete [-t <type>] [-C <comment>] <name>
    r53 [opts] get [-mc] [-t <type>] <name>
    r53 [opts] list [-mc]
    r53 [opts] configure

    Default values:
      -t <type> = A
      -T <ttl> = 60
      -c <comment> = by r53 on YYYY-mm-dd HH:MM:SS

    The following global opts are available:
      -d <domain>: used to complete <name> as required
      -p <profile>: passed to awscli as --profile <profile>
      -z <zoneid>: passed to awscli as --hosted-zone-id <zoneid>
```

* Add or update (UPSERT = UPDATE or INSERT) an 'A' record.
```
$ r53 update squid.example.com 192.0.2.12
```

* Delete the 'A' record.
```
$ r53 delete squid.example.com
```

* Show the 'A' record.
```
$ r53 get squid.example.com
```

* List all records in the hosted zone. '-c' specifies 'compact output' for jq. '-m' makes output even more compact and human-readable by omitting some of the data fields.
```
$ r53 list -mc
```

### Notes for <name>
<name> parameter can be a FQDN or a relative (partial) domain name.

* A trailing dot is optional.
* A FQDN should be in the hosted zone's domain. Otherwise it is treated as a partial name and the hosted domain is appended to the <name>.
* @ is converted to the hosted domain only when the <name> contains only one character '@'.
