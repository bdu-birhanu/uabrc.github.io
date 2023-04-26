# Long-Term Storage

UAB Long-term storage (LTS) is an S3 object-storage platform hosted at UAB. This storage is designed to hold data that is not currently being used in analysis but should be kept for data sharing, recapitulation purposes, or reused for further analysis in the future. This documentation covers multiple methods for accessing LTS in Windows, Mac, and Linux environments.

<!-- markdownlint-disable MD046 -->
!!! important

    Currently, UAB LTS is only accessible using the UAB Campus Network. If you are off campus and want to access LTS, please use the [UAB Campus VPN](https://www.uab.edu/it/home/tech-solutions/network/vpn). If you are accessing LTS through Cheaha, you do not need to use the VPN, even at home.

<!-- markdownlint-enable MD046 -->

<!-- markdownlint-disable MD046 -->
!!! tip

    [Globus](../transfer/globus.md#long-term-storage-s3-lts-connector) may be used to transfer data with LTS.
<!-- markdownlint-enable MD046 -->

## Terminology

When talking about S3 storage, some terms are different compared to a normal filesystem. This section is here to briefly explain some differences in case you go to other documentation and see these terms instead.

- `object`: any unit (i.e. file) stored in LTS. Typical folders and files do not exist in S3.
- `bucket`: The root which objects are stored in. Each account can create a certain number of buckets and each bucket can be shared individually with other users. Bucket names are unique across the entire LTS platform (see the section on [duplicate names](#avoiding-duplicate-names-for-buckets))
- `prefix`: Used in place of a file path to an object, and so can be used to represent an object's place in a typical filesystem. Stored as metadata in each object, prefixes are used for searches in a bucket.
- `policy`: sets permissions for whole buckets and individual objects. Policies allow or deny access to buckets for individual accounts. These are controlled by the owner of the bucket.

This documentation will use the standard file and path terms since those are more easily understood by most users. Just be aware that documentation such as [AWS CLI](https://awscli.amazonaws.com/v2/documentation/api/latest/index.html) will use terms prefix, object, and others that are not standard in a typical filesystem.

## Requesting an Account

UAB researchers do not have automatic access to LTS, and currently, single sign on is not enabled. To request access to LTS, please send an email to <support@listserv.uab.edu>. You will be then be given an Access Key and a Secret Access Key, both of which will be used later on. Keep track of both of these keys and do not share them with anyone else, these are your login credentials for LTS.

## Avoiding Duplicate Names for Buckets

Bucket names are shared across all LTS. This means you cannot create a bucket with a name that has already been created by someone else, even if that bucket is not shared with you. When creating bucket names, make them specific and/or unique. For example, davislab for storing data for the entire Davis lab or the name of a specific dataset that is being stored. Do not make names like trial or my-storage.

Good practice when naming buckets is to use a short, descriptive and memorable name, then append a universally unique identifier (UUID) to the end. Websites like <https://www.uuidgenerator.net/> may be used to generate and copy UUIDs. There are $5.3\times 10^{36}$ possible UUIDs, which means the chance of duplicating one is virtually zero. Please see [Wikipedia](https://en.wikipedia.org/wiki/Universally_unique_identifier#Collisions) for math supporting the low rate of duplication.