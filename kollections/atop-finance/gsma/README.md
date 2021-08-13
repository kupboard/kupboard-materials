# ATOP GSMA Mobile Money Kollection

This GSMA Mobile Money kollection is a package for Kupboard to develop and test GSMA Mobile Money API based financial service. In addition to Kupboard, you can deploy this on any cloud for building and testing financial service on GSMA Mobile Money.

You can find several use cases in [**User Guide**](user-guide.md).

## Install using Kupboard

- The name of cluster on which GSMA package is installed must be `finance`.
- Put `atop-finance-gsma-<version>.tag.gz` in `data/kollections`

### Install

```
$ kupboard kollection package -c atop-finance-gsma-<version> -n gsma
```

### Delete

```
$ kupboard kollection package -c atop-finance-gsma-<version> -n gsma -a delete
```

## Install using docker-compose

### Install

```
$ cd packages/gsma/gsma-docker
$ docker-compose up -d
```

### Delete

```
$ cd packages/gsma/gsma-docker
$ docker-compose down
```
