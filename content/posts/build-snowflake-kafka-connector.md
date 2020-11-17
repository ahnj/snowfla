---
title: "Build & Test Snowflake Kafka Connector Plug-in"
date: 2020-11-15T09:11:06-08:00
draft: false

description: "snowflake kafka plug-in build"
featured_image: "/images/post_test_history_view.png"
tags: ["kafka connect", "snowflake", "tests", "key-pair-authentication"]

---

Recently, I found myself in a position of having to develop some degree of expertise with Snowflake Kafka Connect sink [plug-in](https://github.com/snowflakedb/snowflake-kafka-connector).  While this open-source project has some configuration [documentation](https://docs.snowflake.com/en/user-guide/kafka-connector.html), and a [README](https://github.com/snowflakedb/snowflake-kafka-connector/blob/master/README-TEST.md), it was not obvious to me how to initiate a build or get one of the tests up and running.

Step-by-step instructions below demonstrates Snowflake Kafka Connector end-to-end test running on a macOS against a freebie Snowflake [trial](https://signup.snowflake.com/) instance.  Among the menu of end-to-end tests available, I've chosen the [Apache End2End Test AWS](https://github.com/snowflakedb/snowflake-kafka-connector/blob/master/.github/workflows/End2EndTestApacheAws.yml) as it fulfills my goal of testing a particular version of a plug-in within a specific Apache Kafka version stack.  Here are the prerequisites:

* Snowflake instance
* macOS host

# Create private & public keys for Snowflake access
Follow the [instructions](https://docs.snowflake.com/en/user-guide/snowsql-start.html#using-key-pair-authentication) from Snowflake to create the following key files:

* `rsa_key.pub`
* `rsa_key.p8`

Keep your private key passphrase in a environment variable.  You'll need this later to decrypt your p8 key.

```bash
export SNOWSQL_PRIVATE_KEY_PASSPHRASE=human*readable*passphrase
```

# Update Snowflake `user` with public key

Log into Snowflake [instance](https://ph65577.us-east-2.aws.snowflakecomputing.com/) via your web browser, and update your user's `RSA_PUBLIC_KEY` with the key string from file `rsa_key.pub`.

```sql
use role ACCOUNTADMIN;
-- this mile long key string comes from file rsa_key.pub (remove the header/footer and carriage returns)
ALTER USER jimahn SET RSA_PUBLIC_KEY='MIIBIjANBg...';
```

![alter_user_web_gui](/images/alter_user_web_gui.png)


# Test your keys via `snowsql`

My `~/.snowsql/config` file contains a section like below.  If you do not have `snowsql` installed, please see this [post](https://snowfla.com/posts/snowsql-cli-example-config/).

```
...
[connections.training]
accountname = ph65577.us-east-2.aws
username = jimahn
warehouse = COMPUTE_WH
rolename = PUBLIC
database = DEMO_DB
schemaname = PUBLIC
...
```

Invoke `snowsql` with the connection labeled `training` and your private key file path set to `rsa_key.p8`.  When prompted, enter `human*readable*passphrase`.  This will allow Snowflake to decrypt your `rsa_key.p8` key and grant you entry.

```bash
$ snowsql -c training --private-key-path rsa_key.p8
Private Key Passphrase: 
* SnowSQL * v1.2.10
Type SQL statements or !help
jimahn#COMPUTE_WH@DEMO_DB.PUBLIC>
```


Snowflake user and secret setup is now complete.  Let's now embed this into our testing rig.



# Arm Tests with Snowflake secrets
Make a copy of [profile.json.example](https://github.com/snowflakedb/snowflake-kafka-connector/blob/master/profile.json.example) as `profile.json` and enter your Snowflake secrets.

```bash
git clone https://github.com/snowflakedb/snowflake-kafka-connector.git
cd snowflake-kafka-connector
cp profile.json.example .github/scripts/profile.json
vi .github/scripts/profile.json  # enter your secrets into a plain text file
```

Mine looks like below:

```
{
  "user": "jimahn",
  "private_key": "MIIEvgIBADA..3yFZ",
  "encrypted_private_key": "MIIE6TAbB..3o",
  "private_key_passphrase": "human*readable*passphrase",
  "host": "ph65577.us-east-2.aws.snowflakecomputing.com:443",
  "schema": "PUBLIC",
  "database": "DEMO_DB",
  "warehouse": "COMPUTE_WH"
}

```

`private_key` is simply the single-line version of the super-long string we created in file `rsa_key.p8`.

There's an odd Java related(?) twist, which requires yet another version of the key as `encrypted_private_key`.  This variant of the key can be created via:

```bash
pkcs8 -in rsa_key.p8 -outform pem -out rsa_key.pem
```

In summary `profile.json` requires two different variants the of same private key:

* `encrypted_private_key` from rsa_key.p8 file
* `private_key` from rsa_key.pem file

# Encrypt `profile.json`

Now that we have file `profile.json` prepared, we must encrypt it.  This is how secrets are stored within the github repo such that it can then be shared with the tests housed in github `actions`.  Somewhat tedious, but without this step, the private keys would end up being checked-in naked into github.

# Install `gpg`

```bash
brew install gnupg
```

```bash
export SNOWFLAKE_TEST_PROFILE_SECRET=human*readable*passphrase
gpg -c .github/scripts/profile.json  # generate profile.json.gpg
.github/scripts/decrypt_secret.sh aws  # see output in cwd 
rm .github/scripts/profile.json  # clean up
rm profile.json
```

# Test Execution

After all that, we are now finally ready to get the tests launched.  Since the tests are configured to run via github actions, we utilize a nifty utility `act` to help us execute them locally.


# Install `act`
```bash
$ brew install act
```

```bash
$ act -l # list jobs available
                                                                                  
ID                Stage  Name              
build             0      build  
build             0      build             
build             0      build             
build             0      build             
build             0      build             
build             0      build             
$ 
```

Unfortunately, all five jobs are named `build`.  Let's rename the desired job from `build` to `build_apache_aws`:

```bash
sed -i '' 's/build\:/build_apache_aws\:/g' .github/workflows/End2EndTestApacheAws2.yml
```

```bash
$ act -l

ID                Stage  Name              
build_apache_aws  0      build_apache_aws  
build             0      build             
build             0      build             
build             0      build             
build             0      build             
build             0      build             
$
``` 

Now we can run our test as job `build_apache_aws`!

```bash
$ act -v -j build_apache_aws -P ubuntu-18.04=nektos/act-environments-ubuntu:18.04 -s SNOWFLAKE_TEST_PROFILE_SECRET=human*readable*passphrase
```

# Successful Build & Test

Job `build_apache_aws` has two primary objectives, first is a build of the plug-in jar: `snowflake-kafka-connector-1.5.0.jar`, followed by a test of the that jar. Successful build of the jar looks like this:

![cli](/images/plug-in-build-jar.png)

From Snowflake web interface, you should be able to see all of the snowflake operations in action while the test is in flight.

![gui](/images/post_test_history_view.png)



Docker container remains available after the test completion for further inspection.

```bash
$ docker ps
CONTAINER ID        IMAGE                                  COMMAND                  CREATED             STATUS              PORTS               NAMES
7d41fc90f1d3        nektos/act-environments-ubuntu:18.04   "/usr/bin/tail -f /d…"   10 hours ago        Up 10 hours                             act-Kafka-Connector-Apache-End2End-Test-AWS-build-apache-aws
$ docker exec -it 7d41fc90f1d3 /bin/bash                                    
root@docker-desktop:/github/workspace# 
``` 



Lastly, to make all the tests pass with the above setup, I had to remove test `testFIPS` from the test suite.  If you know how to remedy this, please share it with me.

![skipped_test_fips](/images/skipped_test_fips.png)



