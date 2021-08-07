# ATOP PSD2 Open Banking Collection

This PSD2 Open Banking collection is a package for Kupboard to develop and test a financial service based on Open API specification of PSD2 Open Banking. In addition to Kupboard, you can deploy this on any cloud for building and testing financial service on PSD2 Open Banking specification.

You can find several use cases in [**User Guide**](user-guide.md).

## Install using Kupboard

- The name of cluster on which PSD2 package is installed must be `finance`.
- Put `atop-finance-psd2-<version>.tag.gz` in `data/collections`


### Install

```
$ kupboard collection package -c atop-finance-psd2-<version> -n psd2
```

### Delete

```
$ kupboard collection package -c atop-finance-psd2-<version> -n psd2 -a delete
```

## Install using docker-compose

### Install

```
$ cd packages/psd2/psd2-docker
$ docker-compose up -d
```

### Delete

```
$ cd packages/psd2/psd2-docker
$ docker-compose down
```
