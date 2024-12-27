# OWASP CRS - Antivirus Plugin

## Description of Mechanics

The Antivirus Plugin operates by scanning uploaded files or request bodies for malware using ClamAV, integrated via a Lua script. Key variables like `tx.antivirus-plugin_enabled` control the plugin's activation, while `tx.antivirus-plugin_scan_uploaded_file` and `tx.antivirus-plugin_scan_request_body` determine whether files or request bodies are scanned. Files temporarily stored in `FILES_TMPNAMES` are processed in chunks defined by `tx.antivirus-plugin_clamav_chunk_size_bytes`. If a virus is detected, the name is stored in `tx.antivirus-plugin_virus_name`, and the flag `tx.block_malware` is set to block the request. The plugin tracks malicious activity per IP using `ip.malware_counter`, and if it exceeds `tx.malware_burst_counter`, the IP is blocked for `tx.block_malware_timeout`.

## Prerequisities

 * ModSecurity compiled with Lua support
 * LuaSocket library
 * ModSecurity `SecTmpSaveUploadedFiles` directive is `On` for modsecurity vr2 or
   `SecUploadKeepFiles` directive is set to either `On`

## How to determine whether you have Lua support in ModSecurity

Most modern distro packages come with Lua support compiled in. If you are unsure, or if you get odd error messages (e.g. `EOL found`) chances are you are unlucky. To be really sure look for ModSecurity announce Lua support when launching your web server.

```
...
... ModSecurity: LUA compiled version="Lua 5.3"
...
```

## LuaSocket library installation

LuaSocket library should be part of your linux distribution. Here is an example
of installation on Debian linux:  
`apt install lua-socket`

## Plugin installation

For full and up to date instructions for the different available plugin
installation methods, refer to [How to Install a Plugin](https://coreruleset.org/docs/concepts/plugins/#how-to-install-a-plugin)
in the official CRS documentation.

## Configuration

All settings can be done in file `plugins/antivirus-config.conf`.

### Main configuration

#### tx.antivirus-plugin_scan_uploaded_file

This setting can be used to disable or enable antivirus scanning of uploaded
files (FILES_TMPNAMES variable).

Values:
 * 0 - disable antivirus scanning of uploaded files
 * 1 - enable antivirus scanning of uploaded files

Default value: 1

#### tx.antivirus-plugin_scan_request_body

This setting can be used to disable or enable antivirus scanning of request
bodies (REQUEST_BODY variable). Be carefull while enabling this feature as it
may use lots of system resources (depends on your usecase and environment).

Values:
 * 0 - disable antivirus scanning of request bodies
 * 1 - enable antivirus scanning of request bodies

Default value: 0

#### tx.antivirus-plugin_max_data_size_bytes

Maximum data size, in bytes, which are scanned. If data are bigger, the request
is allowed and error is logged into web server error log.

Default value: 1048576

#### tx.antivirus-plugin_network_timeout_seconds

Timeout, in seconds, for connection to antivirus. If this timeout is reached,
the request is allowed and error is logged into web server error log.

Default value: 2

### ClamAV configuration

#### tx.antivirus-plugin_clamav_connect_type

Connection to ClamAV antivirus can be done either by unix socket file or
TCP/IP.

Values:
 * socket - connect using unix socket file
 * tcp - connect using TCP/IP

Default value: socket

#### tx.antivirus-plugin_clamav_socket_file

You need to set full path to the unix socket file if
`tx.antivirus-plugin_clamav_connect_type` is set to `socket`.

Default value: /var/run/clamav/clamd.ctl

#### tx.antivirus-plugin_clamav_address

You need to set IP address or hostname if
`tx.antivirus-plugin_clamav_connect_type` is set to `tcp`.

Default value: 127.0.0.1

#### tx.antivirus-plugin_clamav_port

You need to set port if `tx.antivirus-plugin_clamav_connect_type` is set to
`tcp`.

Default value: 3310

#### tx.antivirus-plugin_clamav_chunk_size_bytes

Data are not send all at once into ClamAV but are splitted into chunks. Using
this setting, you can set the chunk size, in bytes. Make sure that this setting
does not exceed `StreamMaxLength` as defined in ClamAV configuration file
`clamd.conf`.

Default value: 4096

## Testing

Updating

## Virus signatures

Any antivirus solution is useless without good virus signatures. Below is a list
of virus signatures suitable for protection of web applications.

| *Antivirus Software* | *URL* | *Type* | *Note* |
|----------------------|-------|--------|--------|
| ClamAV               | https://www.rfxn.com/projects/linux-malware-detect/ | PHP malware | free of charge |
| ClamAV               | https://malware.expert/signatures/ | PHP malware | commercial |

## License

Please see the enclosed LICENSE file for full details.
