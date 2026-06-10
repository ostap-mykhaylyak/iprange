# iprange

Pure-PHP reimplementation of [iprange(1)](https://manpages.ubuntu.com/manpages/resolute/man1/iprange.1.html):
merge, optimize and compare IP address lists, with **IPv6 support** on top
(the original is IPv4 only).

**Single executable, zero dependencies.** Everything lives in
[`iprange`](iprange): no `require`, no composer, no extensions (no GMP, no
bcmath). Built for environments where the native `iprange` binary cannot be
installed (shared hosting, minimal containers). Requires PHP ≥ 8.5, CLI only.

## Installation

Copy the `iprange` file wherever you need it.

```bash
curl -O https://raw.githubusercontent.com/ostap-mykhaylyak/iprange/main/iprange
chmod +x iprange
./iprange --help          # or: php iprange --help
```

## Features

- **CIDR aggregation**: reads single IPs, CIDR blocks (`1.2.3.0/24`) and
  ranges (`1.1.1.1-1.1.1.10`), outputs the minimal equivalent set of CIDR prefixes.
- **Set operations**: union (default), intersection (`--common`),
  difference (`--exclude-next`).
- **File comparison**: `--compare`, `--compare-first`, `--compare-next`
  with CSV output.
- **DNS resolution**: hostnames in the input are resolved to A and AAAA
  records (disable with `--no-dns`).
- **ipset optimization**: `--ipset-reduce` lowers the entry count by
  allowing a percentage of extra IPs (for `ipset` of type `hash:net`).
- **IPv4 and IPv6**, handled as separate families in every operation.

## Usage

```bash
# merge multiple lists -> minimal CIDRs
./iprange blocklist1.txt blocklist2.txt > merged.txt

# reads stdin when no file is given
cat list.txt | ./iprange

# IPs present in both lists
./iprange list1.txt --common list2.txt

# remove the whitelist from the blocklist
./iprange blocklist.txt --exclude-next whitelist.txt

# count unique IPs
./iprange -C list.txt

# output as ranges or single IPs
./iprange --print-ranges list.txt
./iprange --print-single-ips list.txt

# no prefix shorter than /24 (larger blocks are split)
./iprange --min-prefix 24 list.txt

# optimize for ipset hash:net, allowing up to 20% extra IPs
./iprange --ipset-reduce 20 --ipset-reduce-entries 16384 list.txt

# compare lists pairwise (CSV)
./iprange --compare list1.txt list2.txt list3.txt
```

Input format: one or more entries per line, separated by spaces or commas;
blank lines and `#` comments are ignored; accepts IPs, CIDRs, `a-b` ranges
and hostnames (IPv4 and IPv6 freely mixed).

## Differences from iprange(1)

- **IPv6 supported** (the original is IPv4 only); `--ipset-reduce` remains
  IPv4 only as in the original, IPv6 passes through unchanged.
- `--ipset-reduce` uses a greedy heuristic with the same contract as the
  original (never more than PERCENT% extra IPs, never below N entries):
  output is always valid but merge choices may differ.
- The `--compare*` CSV output is equivalent in content but not
  byte-identical.
- Not implemented: `--dns-threads` (resolution is sequential) and the
  hex/binary print options.

## License

[MIT](LICENSE)
