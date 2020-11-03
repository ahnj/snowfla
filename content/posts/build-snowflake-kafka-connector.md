---
title: "Build Snowflake Kafka Connector"
date: 2020-11-01T09:11:06-08:00
draft: true

description: "snowflake kafka plug-in tests"
featured_image: "/images/reverse.png"
tags: ["kafka connect", "snowflake", "tests"]

---

Recently, I found myself in a position having to develop some degree of expertise with Snowflake Kafka Connect sink [plug-in](https://github.com/snowflakedb/snowflake-kafka-connector).  While the product itself has some minimal configuration [documentation](https://docs.snowflake.com/en/user-guide/kafka-connector.html), and a [README-TEST](https://github.com/snowflakedb/snowflake-kafka-connector/blob/master/README-TEST.md) it was not obvious to me where to start or how.

Below are the step-by-step instructions to get AWS Snowflake end-to-end tests running on your macOS laptop.  Among the six tests available, the [End2EndTest.yml](https://github.com/snowflakedb/snowflake-kafka-connector/blob/master/.github/workflows/End2EndTest.yml) is the one that satisfies my goal of testing against a particilar Kafka Apache version. Here are the prerequistes:

* Snowflake account key/secret
* AWS secrets
* macOS machine, beefier the better

# Run Unit tests
```bash
mvn package -Dgpg.skip=true
```

# Install pgp

```bash
brew install gnupg
```

# Provide Snowflake and AWS secrets
Make a copy of [profile.json.example] as `profile.json` and enter your aws and Snowflake secrets.  

```bash
git clone https://github.com/snowflakedb/snowflake-kafka-connector.git
cd snowflake-kafka-connector
pushd
cp profile.json.example .github/scripts/profile.json
vi profile.json  # enter your secrets into a plain text file
gpg -c profile.json  # enter SNOWFLAKE_TEST_PROFILE_SECRET to generate profile.json.gpg
export SNOWFLAKE_TEST_PROFILE_SECRET=*enter*your*secret*here
popd
.github/scripts/decrypt_secret.sh aws  # output a descripted copy of profile.json in current directory 
rm .github/scripts/profile.json
```


# install OpenJDK
```bash
brew install openjdk
```

# install helm

```bash
brew install helm
```

# install minikube

```bash
brew install minikube
```

# install jq

```bash
brew install jq
```

# install mvn
```
brew install maven
```
