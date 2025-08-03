---
title: Manual
weight: 1
tags:
  - ddnsd
comments: false
---

## NAME

ddnsd - Dynamic DNS (Domain Name Service) Daemon

## SYNOPSIS

_ddnsd_ ( start | stop | restart | status | version | help )

## DESCRIPTION

_ddnsd_ is a client for updating dynamic DNS records for a domain hosted on _Cloudflare_ or a subdomain hosted on _DuckDNS_. It can be used to update the IP address of a domain or subdomain when the IP address of the host changes. This is useful if your internet service provider doesn't provide a static IP address.

_start_
: Starts the DDNSD client service. This will run the client in the background and update the DNS records periodically.

_stop_
: Stops the DDNSD client service. This will stop the client from running in the background and updating the DNS records.

_restart_
: Restarts the DDNSD client service. This will stop the client and start it again in the background.

_status_
: Displays the status of the DDNSD client service. This will show whether the client is running or not, and if it is running, how long it has been running.

_config_ [ cloudflare | duckdns | -h | --help ]
: Edit the configuration file for the DDNSD client. This command allows you to set up or modify the settings for either Cloudflare or DuckDNS. See **CONFIG OPTIONS** below for more details.

_version_
: Displays the version of the DDNSD client.

_help_
: Displays the help message for the DDNSD client. This will show the available commands and options.

### CONFIG OPTIONS

The configuration file for the DDNSD client is located at `/etc/ddnsd.conf`. This file is not able to be edited directly, however the configuration that is used by the client can be modified with following options:

_cloudflare_
: Saves the configuration for Cloudflare as cloudflare.conf in the `/etc/ddnsd.conf.d/` directory. This file contains the settings for updating DNS records.

_duckdns_
: Saves the configuration for DuckDNS as duckdns.conf in the `/etc/ddnsd.conf.d/` directory. This file contains the settings for updating subdomain records.

_h_ or _--help_
: Displays the help message for the configuration options. This will show the available options and their descriptions

## SEE ALSO

_ddnsd.conf(5)_, _systemd(1)_

## COPYRIGHT

Copyright (c) under the terms of the [GPL-3.0](https://www.gnu.org/licenses/gpl-3.0.en.html) license.
