---
title: Bypassing Server-Side Request Forgery filters by abusing a bug in Ruby's native resolver
author: EdOverflow
url: https://edoverflow.com/2017/ruby-resolv-bug/
date_published: 2017-11-09
date_scraped: 2026-11-16
platform: personal-blog
disclosed_report: https://hackerone.com/reports/215105
bounty: not stated
vuln_class: ssrf
---

<!-- Raw scraped text below. Public, non-paywalled content only.
     This file is source material, never copied into skills verbatim. -->
## Summary

This is a security advisory for a bug that I discovered in `Resolv::getaddresses` that enabled me to bypass multiple Server-Side Request Forgery filters. Applications such as GitLab and HackerOne were affected by this bug. The disclosure of all reports referenced in this advisory follow HackerOne's Vulnerability Disclosure Guidelines.

This bug was assigned CVE-2017-0904.

## Vulnerability Details

`Resolv::getaddresses` is OS-dependent, therefore by playing around with different IP formats one can return blank values. This bug can be abused to bypass exclusion lists often used to protect against SSRF.

| Machine 1 | Machine 2 |
| --- | --- |
| ruby 2.3.3p222 (2016-11-21) [x86_64-linux-gnu] | ruby 2.3.1p112 (2016-04-26) [x86_64-linux-gnu] |

Machine 1

```
irb(main):002:0> Resolv.getaddresses("127.0.0.1")
=> ["127.0.0.1"]
irb(main):003:0> Resolv.getaddresses("localhost")
=> ["127.0.0.1"]
irb(main):004:0> Resolv.getaddresses("127.000.000.1")
=> ["127.0.0.1"]
```

Machine 2

```
irb(main):008:0> Resolv.getaddresses("127.0.0.1")
=> ["127.0.0.1"]
irb(main):009:0> Resolv.getaddresses("localhost")
=> ["127.0.0.1"]
irb(main):010:0> Resolv.getaddresses("127.000.000.1")
=> []
```

This issue is reproducible in the latest stable build of Ruby:

```
$ ruby -v
ruby 2.4.3p201 (2017-10-11 revision 60168) [x86_64-linux]
$ irb
irb(main):001:0> require 'resolv'
=> true
irb(main):002:0> Resolv.getaddresses("127.000.001")
=> []
```

## Proof of concept

```
irb(main):001:0> require 'resolv'
=> true
irb(main):002:0> uri = "0x7f.1"
=> "0x7f.1"
irb(main):003:0> server_ips = Resolv.getaddresses(uri)
=> [] # The bug!
irb(main):004:0> blocked_ips = ["127.0.0.1", "::1", "0.0.0.0"]
=> ["127.0.0.1", "::1", "0.0.0.0"]
irb(main):005:0> (blocked_ips & server_ips).any?
=> false # Bypass
```

## Root cause

The following section describes the root cause of this bug. I have added some comments in the code snippets to help the reader follow along.

When we run irb in debug mode (`irb -d`) the following error is returned:

```
irb(main):002:0> Resolv.getaddresses "127.1"
Exception `Resolv::DNS::Config::NXDomain' at /usr/lib/ruby/2.3.0/resolv.rb:549 - 127.1
Exception `Resolv::DNS::Config::NXDomain' at /usr/lib/ruby/2.3.0/resolv.rb:549 - 127.1
=> []
```

So the exception stems from `fetch_resource()`. The "NXDOMAIN" response indicates that the resolver cannot find a corresponding PTR record. No surprise there, since, as we will see later on, `resolv.rb` uses the operating system's resolver.

```
# Reverse DNS lookup on Machine 1.
$ nslookup 127.0.0.1
Server:   127.0.0.53
Address:  127.0.0.53#53

Non-authoritative answer:
1.0.0.127.in-addr.arpa  name = localhost.

$ nslookup 127.000.000.1
Non-authoritative answer:
Name: 127.000.000.1
Address: 127.0.0.1

# NXDOMAIN for 127.1.
$ nslookup 127.1
** server can't find 127.1: NXDOMAIN
```

Now the following code snippets demonstrate why `Resolv::getaddresses` is OS-dependent.

`getaddresses` takes the address (`name`) and passes it on to `each_address` where once it has been resolved it is appended to the `ret` array.

```ruby
# File lib/resolv.rb, line 100
def getaddresses(name)
  # This is the "ret" array.
  ret = []
  # This is where "address" is appended to the "ret" array.
  each_address(name) {|address| ret << address}
  return ret
end
```

`each_address` runs the `name` through `@resolvers`.

```ruby
# File lib/resolv.rb, line 109
def each_address(name)
    if AddressRegex =~ name
      yield name
      return
    end
    yielded = false
    # "name" is passed on to the resolver here.
    @resolvers.each {|r|
      r.each_address(name) {|address|
        yield address.to_s
        yielded = true
      }
      return if yielded
    }
end
```

`@resolvers` is initialised in `initialize()`.

```ruby
# File lib/resolv.rb, line 109
def initialize(resolvers=[Hosts.new, DNS.new])
    @resolvers = resolvers
end
```

Further on, `initialize` is actually initialised by setting `config_info` to `nil` which uses the default configuration in this case `/etc/resolv.conf`.

```ruby
# File lib/resolv.rb, line 308
def initialize(config_info=nil)
  @mutex = Thread::Mutex.new
  @config = Config.new(config_info)
  @initialized = nil
end
```

Here is the default configuration:

```ruby
# File lib/resolv.rb, line 959
def Config.default_config_hash(filename="/etc/resolv.conf")
  if File.exist? filename
    config_hash = Config.parse_resolv_conf(filename)
  else
    if /mswin|cygwin|mingw|bccwin/ =~ RUBY_PLATFORM
      require 'win32/resolv'
      search, nameserver = Win32::Resolv.get_resolv_info
      config_hash = {}
      config_hash[:nameserver] = nameserver if nameserver
      config_hash[:search] = [search].flatten if search
    end
  end
  config_hash || {}
end
```

This demonstrates that `Resolv::getaddresses` is OS-dependent and that `getaddresses` returns an empty `ret` array when supplied with an IP address that fails during a reverse DNS lookup.

## Mitigation

I suggest staying away from `Resolv::getaddresses` altogether and using the `Socket` library.

```
irb(main):002:0> Resolv.getaddresses("127.1")
=> []
irb(main):003:0> Socket.getaddrinfo("127.1", nil).sample[3]
=> "127.0.0.1"
```

The Ruby Core dev team suggested using the same library (quote from Tanaka Akira, paraphrase: check addresses with the OS resolver, for example Addrinfo.getaddrinfo from the socket library, rather than resolv.rb).

```
% ruby -rsocket -e '
as = Addrinfo.getaddrinfo("192.168.0.1", nil)
p as
p as.map {|a| a.ipv4_private? }
'
[#<Addrinfo: 192.168.0.1 TCP>, #<Addrinfo: 192.168.0.1 UDP>, #<Addrinfo: 192.168.0.1 SOCK_RAW>]
[true, true, true]
```

## Affected Applications and gems

### GitLab Community Edition and Enterprise Edition

Link to report: https://hackerone.com/reports/215105

The fix for Mustafa Hasan's report (https://hackerone.com/reports/135937, GitLab issue !17286) could be easily bypassed by abusing this bug. GitLab introduced an exclusion list, but would resolve the user-supplied address using `Resolv::getaddresses` and then compare the output to the values in the exclusion list. This meant that one could no longer use certain addresses such as `http://127.0.0.1` and `http://localhost/`, which Mustafa Hasan used in the original report. The bypasses allowed me to scan a GitLab instance's internal network.

[images: error message for open port and closed port]

GitLab have provided a patch: https://about.gitlab.com/2017/11/08/gitlab-10-dot-1-dot-2-security-release/.

### private_address_check by John Downey

Link to report: https://github.com/jtdowney/private_address_check/issues/1

private_address_check is a Ruby gem that helps prevent SSRF. The actual filtering takes place in `lib/private_address_check.rb`. The process starts by attempting to resolve the user-supplied URL with `Resolv::getaddresses` and then compares the returned value with the values in the blacklist. Once again I was able to use the same technique as before with GitLab to bypass this filter.

```ruby
# File lib/private_address_check.rb, line 32
def resolves_to_private_address?(hostname)
  ips = Resolv.getaddresses(hostname)
  ips.any? do |ip|
    private_address?(ip)
  end
end
```

Consequently, HackerOne was affected by this bypass (https://hackerone.com/reports/287245), because they use the private_address_check gem to prevent SSRF on the "Integrations" panel: `https://hackerone.com/{BBP}/integrations`.

[images: blocked address, bypass]

Unfortunately, I was unable to exploit this SSRF and therefore the issue only consisted of a filter bypass. HackerOne still encouraged me to report it, because they take any potential security issue into consideration and this bypass demonstrated a potential risk.

This issue was patched in version 0.4.0.

## Unaffected applications and gems

### ssrf_filter by Arkadiy Tetelman

This gem is not vulnerable, because it checks if the value returned is empty.

```ruby
# File lib/ssrf_filter/ssrf_filter.rb, line 116
raise UnresolvedHostname, "Could not resolve hostname '#{hostname}'" if ip_addresses.empty?
```

```
irb(main):002:0> SsrfFilter.get("http://127.1/")
SsrfFilter::UnresolvedHostname: Could not resolve hostname '127.1'
```

### faraday-restrict-ip-addresses by Ben Lavender

This gem uses `Addrinfo.getaddrinfo` as recommended by the Ruby Core dev team.

```ruby
# File lib/faraday/restrict_ip_addresses.rb, line 61
def addresses(hostname)
      Addrinfo.getaddrinfo(hostname, nil, :UNSPEC, :STREAM).map { |a| IPAddr.new(a.ip_address) }
    rescue SocketError => e
      # In case of invalid hostname, return an empty list of addresses
      []
end
```

## Conclusion

The author would like to acknowledge the help provided by Tom Hudson and Yasin Soliman during the discovery of the bug.

Both John Downey and Arkadiy Tetelman were extremely responsive. John Downey was able to immediately provide a patch, and Arkadiy Tetelman helped me figure out why their gem was not affected by the issue.

Update (Friday, 10 November 2017): I expanded the "Root cause" section in order to better explain the actual issue.
