---
title: azcopy sync | Microsoft Docs
description: This article provides reference information for the azcopy sync command.
author: normesta
ms.service: storage
ms.topic: reference
ms.date: 09/01/2021
ms.author: normesta
ms.subservice: common
ms.reviewer: zezha-msft
---

# azcopy sync

Replicates the source location to the destination location. This article provides a detailed reference for the azcopy sync command. To learn more about synchronizing blobs between source and destination locations, see [Synchronize with Azure Blob storage by using AzCopy v10](storage-use-azcopy-blobs-synchronize.md). For Azure Files, see [Synchronize files](storage-use-azcopy-files.md#synchronize-files).

## Synopsis

The last modified times are used for comparison. The file is skipped if the last modified time in the destination is more recent.

The supported pairs are:

- local <-> Azure Blob (either SAS or OAuth authentication can be used)
- Azure Blob <-> Azure Blob (Source must include a SAS or is publicly accessible; either SAS or OAuth authentication can be used for destination)
- Azure File <-> Azure File (Source must include a SAS or is publicly accessible; SAS authentication should be used for destination)

The sync command differs from the copy command in several ways:

1. By default, the recursive flag is true and sync copies all subdirectories. Sync only copies the top-level files inside a directory if the recursive flag is false.
2. When syncing between virtual directories, add a trailing slash to the path (refer to examples) if there's a blob with the same name as one of the virtual directories.
3. If the `deleteDestination` flag is set to true or prompt, then sync will delete files and blobs at the destination that are not present at the source.

## Related conceptual articles

- [Get started with AzCopy](storage-use-azcopy-v10.md)
- [Tutorial: Migrate on-premises data to cloud storage with AzCopy](storage-use-azcopy-migrate-on-premises-data.md)
- [Transfer data with AzCopy and Blob storage](./storage-use-azcopy-v10.md#transfer-data)
- [Transfer data with AzCopy and file storage](storage-use-azcopy-files.md)

### Advanced

If you don't specify a file extension, AzCopy automatically detects the content type of the files when uploading from the local disk, based on the file extension or content (if no extension is specified).

The built-in lookup table is small, but on Unix, it's augmented by the local system's mime.types file(s) if available under one or more of these names:

- /etc/mime.types
- /etc/apache2/mime.types
- /etc/apache/mime.types

On Windows, MIME types are extracted from the registry.

```azcopy
azcopy sync <source> <destination> [flags]
```

## Examples

Sync a single file:

```azcopy
azcopy sync "/path/to/file.txt" "https://[account].blob.core.windows.net/[container]/[path/to/blob]"
```

Same as above, but also compute an MD5 hash of the file content, and then save that MD5 hash as the blob's Content-MD5 property.

```azcopy
azcopy sync "/path/to/file.txt" "https://[account].blob.core.windows.net/[container]/[path/to/blob]" --put-md5
```

Sync an entire directory including its subdirectories (note that recursive is by default on):

```azcopy
azcopy sync "/path/to/dir" "https://[account].blob.core.windows.net/[container]/[path/to/virtual/dir]"
```

or

```azcopy
azcopy sync "/path/to/dir" "https://[account].blob.core.windows.net/[container]/[path/to/virtual/dir]" --put-md5
```

Sync only the files inside of a directory but not subdirectories or the files inside of subdirectories:

```azcopy
azcopy sync "/path/to/dir" "https://[account].blob.core.windows.net/[container]/[path/to/virtual/dir]" --recursive=false
```

Sync a subset of files in a directory (For example: only jpg and pdf files, or if the file name is `exactName`):

```azcopy
azcopy sync "/path/to/dir" "https://[account].blob.core.windows.net/[container]/[path/to/virtual/dir]" --include-pattern="*.jpg;*.pdf;exactName"
```

Sync an entire directory but exclude certain files from the scope (For example: every file that starts with foo or ends with bar):

```azcopy
azcopy sync "/path/to/dir" "https://[account].blob.core.windows.net/[container]/[path/to/virtual/dir]" --exclude-pattern="foo*;*bar"
```

Sync a single blob:

```azcopy
azcopy sync "https://[account].blob.core.windows.net/[container]/[path/to/blob]?[SAS]" "https://[account].blob.core.windows.net/[container]/[path/to/blob]"
```

Sync a virtual directory:

```azcopy
azcopy sync "https://[account].blob.core.windows.net/[container]/[path/to/virtual/dir]?[SAS]" "https://[account].blob.core.windows.net/[container]/[path/to/virtual/dir]" --recursive=true
```

Sync a virtual directory that has the same name as a blob (add a trailing slash to the path in order to disambiguate):

```azcopy
azcopy sync "https://[account].blob.core.windows.net/[container]/[path/to/virtual/dir]/?[SAS]" "https://[account].blob.core.windows.net/[container]/[path/to/virtual/dir]/" --recursive=true
```

Sync an Azure File directory:

```azcopy
azcopy sync "https://[account].file.core.windows.net/[share]/[path/to/dir]?[SAS]" "https://[account].file.core.windows.net/[share]/[path/to/dir]?[SAS]" --recursive=true
```

> [!NOTE]
> If include/exclude flags are used together, only files matching the include patterns would be looked at, but those matching the exclude patterns would be always be ignored.

## Options

**--block-size-mb** float    Use this block size (specified in MiB) when uploading to Azure Storage or downloading from Azure Storage. Default is automatically calculated based on file size. Decimal fractions are allowed (For example: `0.25`).

**--check-md5** string   Specifies how strictly MD5 hashes should be validated when downloading. This option is only available when downloading. Available values include: `NoCheck`, `LogOnly`, `FailIfDifferent`, `FailIfDifferentOrMissing`. (default `FailIfDifferent`). (default `FailIfDifferent`)

**--cpk-by-name** string          Client provided key by name let clients making requests against Azure Blob Storage an option to provide an encryption key on a per-request basis. Provided key name will be fetched from Azure Key Vault and will be used to encrypt the data

**--cpk-by-value**                Client provided key by name let clients making requests against Azure Blob Storage an option to provide an encryption key on a per-request basis. Provided key and its hash will be fetched from environment variables

**--delete-destination** string   Defines whether to delete extra files from the destination that are not present at the source. Could be set to `true`, `false`, or `prompt`. If set to `prompt`, the user will be asked a question before scheduling files and blobs for deletion. (default `false`). (default `false`)

**--dry-run**                     Prints the path of files that would be copied or removed by the sync command. This flag does not copy or remove the actual files.

**--exclude-attributes** string   (Windows only) Excludes files whose attributes match the attribute list. For example: `A;S;R`

**--exclude-path** string    Exclude these paths when comparing the source against the destination. This option does not support wildcard characters (*). Checks relative path prefix(For example: `myFolder;myFolder/subDirName/file.pdf`).

**--exclude-pattern** string   Exclude files where the name matches the pattern list. For example: `*.jpg;*.pdf;exactName`

**--exclude-regex** string        Exclude the relative path of the files that match with the regular expressions. Separate regular expressions with ';'.

**--help**    help for sync.

**--include-attributes** string   (Windows only) Includes only files whose attributes match the attribute list. For example: `A;S;R`

**--include-pattern** string   Include only files where the name matches the pattern list. For example: `*.jpg;*.pdf;exactName`

**--log-level** string     Define the log verbosity for the log file, available levels: `INFO`(all requests and responses), `WARNING`(slow responses), `ERROR`(only failed requests), and `NONE`(no output logs). (default `INFO`).

**--mirror-mode**          Disable last-modified-time based comparison and overwrites the conflicting files and blobs at the destination if this flag is set to `true`. Default is `false`.

**--preserve-smb-info**   True by default. Preserves SMB property info (last write time, creation time, attribute bits) between SMB-aware resources (Windows and Azure Files). This flag applies to both files and folders, unless a file-only filter is specified (for example, include-pattern). The info transferred for folders is the same as that for files, except for Last Write Time that is not preserved for folders.

**--preserve-permissions**        False by default. Preserves ACLs between aware resources (Windows and Azure Files, or Data Lake Storage Gen 2 to Data Lake Storage Gen 2). For accounts that have a hierarchical namespace, you will need a container SAS or OAuth token with Modify Ownership and Modify Permissions permissions. For downloads, you will also need the --backup flag to restore permissions where the new Owner will not be the user running AzCopy. This flag applies to both files and folders, unless a file-only filter is specified (e.g. include-pattern).

**--put-md5**     Create an MD5 hash of each file, and save the hash as the Content-MD5 property of the destination blob or file. (By default the hash is NOT created.) Only available when uploading.

**--recursive**    `True` by default, look into subdirectories recursively when syncing between directories. (default `True`).

**--s2s-preserve-access-tier**  Preserve access tier during service to service copy. Refer to [Hot, cool, and archive access tiers for blob data](../blobs/access-tiers-overview.md) to ensure destination storage account supports setting access tier. In the cases that setting access tier is not supported, please use `--s2s-preserve-access-tier=false` to bypass copying access tier. (default `true`).

**--s2s-preserve-blob-tags**      Preserve index tags during service to service sync from one blob storage to another.

## Options inherited from parent commands

|Option|Description|
|---|---|
|--cap-mbps uint32|Caps the transfer rate, in megabits per second. Moment-by-moment throughput might vary slightly from the cap. If this option is set to zero, or it is omitted, the throughput isn't capped.|
|--output-type string|Format of the command's output. The choices include: text, json. The default value is "text".|
|--trusted-microsoft-suffixes string   |Specifies additional domain suffixes where Azure Active Directory login tokens may be sent.  The default is '*.core.windows.net;*.core.chinacloudapi.cn;*.core.cloudapi.de;*.core.usgovcloudapi.net'. Any listed here are added to the default. For security, you should only put Microsoft Azure domains here. Separate multiple entries with semi-colons.|

## See also

- [azcopy](storage-ref-azcopy.md)
