> **Archived.** These scripts are from 2013–2014. They pulled dead Apache nodes out of a tinydns round-robin for a small LAMP cluster.
>
> That is not a good 2026 setup:
> - tinydns is unmaintained. Bernstein released djbdns to the public domain in 2007 and stopped. No DNSSEC, awkward IPv6. Authoritative DNS now is Knot, NSD, or PowerDNS.
> - Round-robin DNS is a poor load balancer. Caches ignore TTL, failback is slow, and every node rewriting the same `data` file is a race. Use a real load balancer, keepalived, or a DNS host with health checks.
> - `checkserver.php` uses the `mysql_*` extension, removed in PHP 7.
>
> Left here as a historical example of cron-driven tinydns failover.

# djbdns-tinydns

Scripts to implement round-robin DNS failover/failback for an Apache cluster. Assumes [djbdns](https://cr.yp.to/djbdns.html) tinydns.

`monitor-nodes.sh` runs from cron on each node. It wget's `checkserver.php` on every cluster IP. After two consecutive failures it rewrites tinydns `data`, turning `+` A records into `-` (disabled), then `make data.cdb`. When the node comes back it flips them back. If every node looks down it leaves DNS alone. On the designated master it may also try a Percona `bootstrap-pxc` if MySQL is gone.

Written for a tuxlite LAMP setup. Last real change March 2014.

## Setup

Crontab on each node:

```
*/1 * * * * /usr/local/bin/monitor-nodes.sh
```

On each cluster node install the check script, e.g. for tuxlite LAMP:

```
/home/<user>/domains/<node>/public_html/checkserver.php
```
