# r53
AWS Route 53 DNS Record Updater

## Prerequisite
* awscli
* jq
* an IAM account which has the following privileges.
  + ChangeResourceRecordSets action (on your hosted zone)
  + ListResourceRecordSets (on your hosted zone)
  + GetChange (on route53)
  + GetHostedZone (on your hosted zones)
  + ListHostedZones (\*)
* an awscli profile which has been configured for use with the IAM account.
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
Run 'r53 configure' and input your awscli profile and a hosted domain information (Zone ID is automatically queried using a input domain name). They are written to your $HOME/.r53rc file and will be used from now on.
```
$ ./r53 configure
AWS CLI Profile [default]: dns
Hosted Zone Domain []: example.com
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
["octopus.example.com.","A",3600,["192.0.2.8"]]
["squid.example.com.","A",60,["192.0.2.12"]]
["www.example.com.","CNAME",3600,["octopus.example.com."]]
["example.com.","NS",172800,["ns-xxxx.awsdns-xx.co.uk.", "ns-xxxx.awsdns-xx.com.", "ns-xxxx.awsdns-xx.net.", "ns-xxxx.awsdns-xx.org."]]
["example.com.","SOA",900,["ns-XXXX.awsdns-xx.co.uk. awsdns-hostmaster.amazon.com. 1 7200 900 1209600 86400"]]
```

* Add a TXT record with TTL = 120.
```
$ r53 update -t txt -T 120 _acme-challenge.example.com xxxxx...
```

### Notes for &lt;name&gt; parameter
&lt;name&gt; parameter can be a FQDN or a relative (partial) domain name.

1. A trailing dot is optional and doesn't mean FQDN. Usually it means that the name is a FQDN. But r53 treats it differently as follows.
2. A FQDN should be in the configured hosted domain. Otherwise it is treated as a partial name and the hosted domain is appended to the &lt;name&gt;.
3. @ is converted to the hosted domain only when the &lt;name&gt; is exactly the same as "@". If the &lt;name&gt; contains any other characters or two or more @s, it is treated as it is.

Those rules can be tested by using 'r53 testname' hidden subcommand.
```
Rule 1: trailing dot 
$ r53 testname salmon.example.com
salmon.example.com.
$ r53 testname salmon.example.com.
salmon.example.com.

Rule 2: names outside of the hosted domain
$ r53 testname tuna
tuna.example.com.
$ r53 testname tuna.
tuna.example.com.
$ r53 testname salmon.example.net.
salmon.example.net.example.com.

Rule 3: only a single @ is regarded as the hosted domain shorthand
$ r53 testname @
example.com.
$ r53 testname mackerel.@
mackerel.@.example.com.
```
