---
title: Using sgadmin
slug: sgadmin
category: configuration
order: 100
layout: docs
description: Use the powerful sgadmin command line tool to manage and configure  everything in Search Guard.
---
<!---
Copryight 2017 floragunn GmbH
-->

# Using sgadmin to configure Search Guard

The Search Guard configuration, including users, roles and permissions, is stored in an index on the Elasticsearch cluster. This allows for hot configuration reloading, and eliminates the need to place configuration files on each node.

Configuration settings are loaded into this Search Guard configuration index using the `sgadmin` tool. `sgadmin` identifies itself against a Search Guard secured Elasticsearch cluster via an admin TLS certificate, either in `.jks` or `.pem` format. An admin certificate grants full access to the cluster, including making changes to the Search Guard configuration index.

## Configuring the admin certificate

You can configure all certificates that should have admin privileges in ``elasticsearch.yml`` by stating their respective DNs. If you use the [example PKI scripts](https://github.com/floragunncom/search-guard-ssl/tree/master/example-pki-scripts) to generate the certificates, you can use the _kirk_ or _spock_ client certificate:

```yaml
searchguard.authcz.admin_dn:
  - CN=kirk,OU=client,O=client,L=test, C=DE
  - CN=spock,OU=client,O=client,L=test, C=DE
```

*Do not use node certificates as admin certifcates. The intended use is to keep node and admin certificates separate. Using node certificates as admin certificates can lead to unexpected results. Also, do not use any whitespaces between the parts of the DN.*

## Basic Usage

The sgadmin CLI tool can be run from any machine that has access to the transport port of your Elasticsearch cluster (default: 9300). This means that you can change the Search Guard configuration without having to access your nodes via SSH. 

Search Guard ships with sgadmin included:

```
<ES installation directory>/plugins/search-guard-5/tools
```

### Linux

Change the permissions on that script and give it execution rights:

```
chmod +x plugins/search-guard-5/tools/sgadmin.sh
```

### Windows

Before executing sgadmin.bat check that you have set JAVA_HOME environment variable, e.g.:

```
set JAVA_HOME=C:\Program Files\Java\jdk1.8.0_65
```

Replace `jdk1.8.0_65` with your installed JDK or JRE version.

### Standalone version

You can also download the sgadmin standalone version. It comes with all dependencies included, so you can use it on any machine you want, provided Java is installed.

* [Download for Search Guard 5 and Elasticsearch 5.5.x](https://search.maven.org/remotecontent?filepath=com/floragunn/search-guard-5/5.5.2-15/search-guard-5-5.5.2-15-sgadmin-standalone.zip){:target="_blank"}
* [Download for Search Guard 5 and Elasticsearch 5.4.x](https://search.maven.org/remotecontent?filepath=com/floragunn/search-guard-5/5.4.3-15/search-guard-5-5.4.3-15-sgadmin-standalone.zip){:target="_blank"}
* [Download for Search Guard 5 and Elasticsearch 5.3.x](https://search.maven.org/remotecontent?filepath=com/floragunn/search-guard-5/5.3.3-15/search-guard-5-5.3.3-15-sgadmin-standalone.tar.gz){:target="_blank"}
* [Download for Search Guard 5 and Elasticsearch 5.2.x](http://search.maven.org/remotecontent?filepath=com/floragunn/search-guard-5/5.2.2-12/search-guard-5-5.2.2-12-sgadmin-standalone.zip){:target="_blank"}
* [Download for Search Guard 5 and Elasticsearch 5.1.x](http://search.maven.org/remotecontent?filepath=com/floragunn/search-guard-5/5.1.2-12/search-guard-5-5.1.2-12-sgadmin-standalone.zip){:target="_blank"}
* [Download for Search Guard 5 and Elasticsearch 5.0.x](http://search.maven.org/remotecontent?filepath=com/floragunn/search-guard-5/5.0.2-12/search-guard-5-5.0.2-12-sgadmin-standalone.zip){:target="_blank"}
* [Download for Search Guard 2 and Elasticsearch 2.4.x](http://search.maven.org/remotecontent?filepath=com/floragunn/search-guard-2/2.4.5.12/search-guard-2-2.4.5.12-sgadmin-standalone.zip){:target="_blank"}

### Print all command line options

To print all available command line options, execute sgadmin without any command line switches:

```
<ES installation directory>/plugins/search-guard-5/tools/sgadmin.sh
```

## Using sgadmin with Keystore and Truststore files

You need to specify the key and truststore file location, their respective passwords, and the directory where the Search Guard configuation files can be found. 

In addition, you either specify the cluster name (`-cn` option), or, as in this example, tell `sgadmin` to ignore the cluster name completely (`-icl` option).

```bash
./sgadmin.sh -ts <path/to/truststore> -tspass <truststore password>  \
  -ks <path/to/keystore> -kspass <keystore password>  \
  -cd ../sgconfig -icl -nhnv
```

The path to the keystore and truststore files are resolved relative to the directory where you execute sgadmin. You can also use absolute paths. This also applies for the configuration directory (-cd option).

If you use the certificates generated by the [demo installation script](quickstart.md), copy the truststore.jks and the kirk-keystore.jks to the tools directory and execute:

```bash
./sgadmin.sh -ts truststore.jks -ks kirk.jks  \
  -cd ../sgconfig -icl - nhnv
```

This will push all configuration files in the sgconfig directory to your cluster. Since the generates keystore and truststore have the default password `changeit`, we can omit the `-kspass` and `-tspass` switches here.

After one or more files are updated, Search Guard will automatically reload the new configuration. There is no need to restart any node.

Use the following options to control the key and truststore settings:

| Name | Description |
|---|---|
| -ks | The location of the keystore containing the admin certificate and all intermediate certificates, if any. You can use an absolute or relative path. Relative paths are resolved relative to the execution directory of sgadmin.|
| -kspass | The password for the keystore.|
| -kst | The key store type, either JKS or PKCS12. If not specified, Search Guard tries to deduct the type from the file extension.|
| -ksalias | The alias of the admin certificate, if any.|
| -ts | The location of the truststore containing the root certificate. You can use an absolute or relative path. Relative paths are resolved relative to the execution directory of sgadmin.|
| -tspass | The password for the truststore.|
| -tst | The trust store type, either JKS or PKCS12. If not specified, Search Guard tries to deduct the type from the file extension.|
| -tsalias | The alias for the root certificate, if any.|

## Using sgadmin with PEM certificates

Instead of using Keystore and Truststore files, sgadmin also works with certificates in PEM format, for example:

```bash
./sgadmin.sh -cd ../sgconfig/ -icl -nhnv -cacert root-ca.pem -cert \
  kirk.crtfull.pem -key kirk.key.pem -keypass password
```

| Name | Description |
|---|---|
| -cert | The location of the pem file containing the admin certificate and all intermediate certificates, if any. You can use an absolute or relative path. Relative paths are resolved relative to the execution directory of sgadmin.|
| -key | The location of the pem file containing the private key of the admin certificate. You can use an absolute or relative path. Relative paths are resolved relative to the execution directory of sgadmin. The key must be in PKCS#8 format.|
| -keypass | The password of the private key of the admin certificate, if any.|
| -cacert | The location of the pem file containing the root certificate. You can use an absolute or relative path. Relative paths are resolved relative to the execution directory of sgadmin.|

## Command line options

sgadmin comes with command line options. Execute `./sgadmin.sh` without any options to list them.

### General

| Name  | Description  |
|---|---|
| -f | fail fast if something goes wrong, no retry |

### Elasticsearch settings

If you run a default Elasticsearch installation, which listens on transport port 9300, and uses `elasticsearch` as cluster name, you can omit the following settings altogether. Otherwise, specify your Elasticsearch settings by using the following switches:

| Name | Description |
|---|---|
| -h | elasticsearch hostname, default: localhost |
| -p | elasticsearch port, default: 9300 (NOT the http port!) |
| -cn | clustername, default: elasticsearch |
| -icl | Ignore clustername. |
| -sniff | Sniff cluster nodes. |
| -arc,--accept-red-cluster | Execute sgadmin even if the cluster state is red. Default: sgadmin won't execute on red cluster state |

Ignore cluster name means that the name of your cluster will not be validated. Sniffing can be used to detected available nodes by using the ES cluster state API. You can read more about this feature in the [Elasticsearch documentation](https://www.elastic.co/guide/en/elasticsearch/client/java-api/current/transport-client.html).

### Certificate validation settings

In addition to the Keystore, Truststore or PEM settings above, use the following options to control certificate validation:

| Name | Description |
|---|---|
| -nhnv | disable-host-name-verification, do not validate hostname.|
| -nrhn | disable-resolve-hostname, do not resolve hostname (Only relevant if -nhnv is not set.).|
|-noopenssl| Do not use OpenSSL even if available (default: use OpenSSL if available)|

### Configuration files settings

The following switches define which configuration file(s) you want to push to Search Guard. You can either push a single file or specify a directory containing one or more configuration files.

| Name | Description |
|---|---|
| -cd | Directory containing multiple Search Guard configuration files. |
| -f | Single config file (cannot be used together with -cd).  |
| -t | File type. |
| -rl | Reload the current configuration and flush the internal cache. |

To upload all configuration files in a directory, use:

```bash
./sgadmin.sh -cd ../sgconfig -ts ... -tspass ... -ks ... -kspass ...
```

If you want to push a single configuration file, use:

```bash
./sgadmin.sh -f ../sg_internal_users.yml -t internalusers  \
    -ts ... -tspass ... -ks ... -kspass ...
```

the filetype must be one of:

* config
* roles
* rolesmapping
* internalusers
* actiongroups

### Cipher settings

Usually you do not need to change with the cipher settings. If you do, use the following switches:

| Name | Description |
|---|---|
| -ec | enabled-ciphers, comma separated list of TLS ciphers.|
| -ep | enabled-protocols, comma separated list of TLS protocols.|

### Backup and Restore

You can download all current configuration files from your cluster with the following command:

```bash
./sgadmin.sh -r -ts ... -tspass ... -ks ... -kspass ...
```

This will dump the currently active Search Guard configuration from your cluster to individual files in the working directory. You can then use these files to upload the configuration again to the same or a different cluster. This is for example useful when moving a PoC to production.

You can also specify the download location with the -cd option:

```bash
./sgadmin.sh -r -h staging.example.com -p 9300  \
    -cd /etc/backup/ -ts ... -tspass ... -ks ... -kspass ...
```

To upload the dumped files to another cluster use:

```bash
./sgadmin.sh -h production.example.com -p 9301 -cd /etc/backup/  \
    -ts ... -tspass ... -ks ... -kspass ...
```

| Name | Description |
|---|---|
| -r | retrieve the current Search Guard configuration from a running cluster, and dump it to configuration files in the working directory|
| -cd | Specify the directory to store the files to. You can use an absolute or relative path. Relative paths are resolved relative to the execution directory of sgadmin.|

### Cache invalidation 

Search Guard by default caches authenticated users and their roles and permissions for one hour. You can invalidate the cache by reloading the Search Guard configuration:

```bash
./sgadmin.sh -rl -ts ... -tspass ... -ks ... -kspass ...
```
| Name | Description |
|---|---|
| -rl | reload the current Search Guard configuration stored in your cluster, invalidating any cached users, roles and permissions.|


### Index and replica settings

The following switched control the Search Guard index settings.

| Name | Description |
|---|---|
| -i | Search Guard index name, defaults to searchguard.|
| -us | Update the replica settings.|
| -era | Enable replica auto-expand.|
| -dra | Disable replica auto-expand.|

The first time you run sgadmin.sh, the ```-us```, ```-era```, ```dra```, and ```-rl``` (reload configuration), flags can cause the initial setup to fail, as the searchguard index does not yet exist.

See chapter [index management](sgindex.md) for more details on how the Search Guard index is structured and how to manage it.