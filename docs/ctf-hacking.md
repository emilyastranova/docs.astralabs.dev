# CTF/Hacking

Learn about CTFs and how to hack like a pro.

## Relaying Traffic

### `socat`

`socat` is a versatile networking utility that allows you to relay traffic between different network endpoints while providing various options for data manipulation and redirection.

Example:

You can use `socat` to relay traffic from one network endpoint to another while sniffing and printing the data to standard output. Here's an example command:

```shell
socat -v TCP-LISTEN:389,bind=127.0.0.1,fork TCP:<legitDC>:389
```

## Extracting and Sorting Password Hashes from Responder Logs

If you need to extract and sort unique password hashes from Responder logs, you can use the following command:

```shell
cat /path/to/Responder/logs/*.txt | sort -k1,1 -t: -u
```

## Recursively Retrieve Files from an SMB Folder

To recursively retrieve files from an SMB (Server Message Block) folder, you can use the following `smbclient` command:

```shell
smbclient '\\server\share' -N -c 'prompt OFF;recurse ON;cd 'path\to\directory\';lcd '~/path/to/download/to/';mget *'
```
