---
title: Configuration
date: 2025-08-01T05:17:45-06:00
draft: true
featuredImage: ""
externalLink: ""
---

## NAME

ddnsd-config - Configuration file for the DDNSD client

## SYNOPSIS

`/usr/bin/ddnsd` [ start | stop | restart | status | version | help ]

## DESCRIPTION

The _ddnsd.conf_ file is the configuration file for the DDNSD client. It contains settings that control how the client operates, including the domain or subdomain to update, the update interval, and the API keys for Cloudflare or DuckDNS.

: Verify the provided is set and valid.

: Verify the domain: this includes ping the domain and checking the DNS records.

: Check that the provided API key is valid and active, this

## CONFIGURATION FILE

The configuration file is located at _/etc/ddnsd.conf_.

## GLOBAL SETTINGS

### IP_LOOKUP

The method to use for looking up the current public IP address. Valid values are _icanhazip_, _ipinfo_ or _ifconfig_. These services return the public IP address of the host running the DDNSD client.

Default: _icanhazip_

: Example: _IP_LOOKUP=ipinfo_

### DDNS_PROVIDER

The DNS provider to use for updates. Valid values are _cloudflare_ or _duckdns_.

: Default: _cloudflare_

: Example: _DDNS_PROVIDER=cloudflare_

### DOMAIN_NAME

The domain or subdomain to update. This is the full domain name, including any subdomains.

: Default:

: Example: _DOMAIN_NAME=example.com_

### API_KEY

The API key for the DNS provider. This is required for authentication when updating the DNS records.

: Default: (empty)

: Example: _API_KEY=your_api_key_here_

## CLOUDFLARE SETTINGS

### CLOUDFLARE_EMAIL

The email address associated with the Cloudflare account.

: Default: (empty)

: Example: _youremail@email.com_

### CLOUDFLARE_METHOD

The method to use for updating the DNS records. Valid values are _token_ or _global_. In most cases, _token_ is preferred as it provides better security and control. Global is used for legacy support.

: Default: _token_

: Example: _CLOUDFLARE_METHOD=token_

### CLOUDFLARE_ZONE_ID

The zone ID for the domain in Cloudflare. This is required for updating the DNS records.

: Default: (empty)

: Example: _CLOUDFLARE_ZONE_ID=your_zone_id_here_

### CLOUDFLARE_PROXY

The proxy setting for Cloudflare. If set to _true_, the DNS records will be proxied through Cloudflare's network, providing additional security with a negligible performance impact. However, this may not be suitable if you are using a _VPN_ service: this applies to if you self-host your own _VPN_ service like _WireGuard_ or _OpenVPN_.

: Default: _false_

: Example: _CLOUDFLARE_PROXY=true_

### CLOUDFLARE_TTL

The time-to-live (TTL) for the DNS records in Cloudflare. This controls how long the DNS records are cached by resolvers. A lower value means the records will be updated more frequently, while a higher value means they will be cached longer.

Values: _60_, _120_, _300_, _600_, _1800_, _3600_, _7200_, _14400_, _28800_, _43200_, _86400_

: Default: _300_ (5 minutes)

: Example: _CLOUDFLARE_TTL=3600_ (1 hour)

## DUCKDNS SETTINGS

### DUCKDNS_INSECURE

The insecure setting for DuckDNS. If set to _true_, the client will not verify the SSL certificate when making requests to DuckDNS. This is not recommended for production use, as it can expose you to security risks.

: Default: _false_

: Example: _DUCKDNS_INSECURE=true_

### DUCKDNS_VERBOSE

The verbose setting for DuckDNS. If set to _true_, the client will log additional information about the update process, which can be useful for debugging.

: Default: _false_

: Example: _DUCKDNS_VERBOSE=true_

## SEE ALSO

_ddnsd_(8)

## COPYRIGHT

Copyright (c) under the terms of the [GPL-3.0](https://www.gnu.org/licenses/gpl-3.0.en.html) license.
