---
title: "Server Side Request Forgery (SSRF) via Analytics Reports"
author: megaman_
url: https://hackerone.com/reports/2262382
date_published: 2023-12-08 (disclosed; reported 2023-11-23)
date_scraped: 2026-11-16
platform: hackerone
disclosed_report: https://hackerone.com/reports/2262382
bounty: "$25,000"
vuln_class: ssrf
---

<!-- Raw scraped text below. Public, non-paywalled content only.
     This file is source material, never copied into skills verbatim. -->

Program: security | State: Resolved | Severity: "critical" 10.0 | Weakness: Server-Side Request Forgery (SSRF)

## Report body (vulnerability_information)

Hello Gents, I would like to report an issue where attackers are able to read internal files via an SSRF vulnerability.


## Proof of concept

+ ███

## Impact

SSRF.

Thanks and have a nice day!

## Summary by team

We recently received a critical server-side request forgery (SSRF) vulnerability report through our bug bounty program. The issue allowed attackers to make internal requests from our application servers by exploiting a lack of output sanitization in an error message. By crafting malicious requests, an attacker could have accessed internal AWS services and obtained temporary credentials. 

Upon receiving the report, we were able to reproduce and verify the issue. We have implemented a fix that is now deployed in production. Below you can see that element[:template] wasn’t properly sanitized in the error message. The value being returned `html_without_layout` is being fed into a library that converts this into a PDF.

Diff: 

```ruby
             
html_without_layout = elements.map do |element|
  case element[:type]
  when 'template'
    path = template_path(element)
    if path
      ApplicationController.new.render_to_string(
        partial: path,
        layout: false,
        assigns: {
          include_css: true,
        },
        locals: {
          query_results:,
          constants: PSR_CONSTANTS,
          retesting_reports: retesting_reports,
          reporters: reporters,
          total_submission_count: total_submission_count,
          in_scope_asset_count: in_scope_asset_count,
          report_asset_count: report_asset_count,
        },
      )  
    else
-    "Missing template for element: #{element[:template]}"
+    "Missing template for element"

```

We have also added regression tests to prevent future occurrences of this vulnerability. 

Our forensic investigation concluded that there is no evidence this issue was exploited prior to the report.

We have rated this vulnerability CVSSv3 10 (Critical) based on the potential impact of exposed credentials: 

CVSS:3.0/AV:N/AC:L/PR:N/UI:N/S:C/C:H/I:H/A:N/CR:H/IR:H/AR:H. 

We want to thank @mega7 for sending in their report; reports like this one are invaluable for us to continue enhancing our security posture. We also appreciate them for stopping at the right point in testing, demonstrating a responsible and ethical approach.

## Summary by researcher

# Report Conclusion:

+ ## The Main Issue:
> I was able to access *AWS Credentials* using an *SSRF* vulnerability by injecting an `<iframe>` tag into `template` element when generating a `PDF` of analytics reports.
+ ## The potential impacts of this vulnerability:
> 1. Ability to read internal files.
> 2. Unauthorized Access to AWS Credentials.
> 3. Manipulating AWS resources.
> 4. Possibility of Data Loss.
> 5. Accounts Takeover.
> 7. Executing Arbitrary Commands.

+ ## What did I do when I found this vulnerability:
> I quickly reported this issue as soon as I found it, then I was trying to find the quickest way to alert Hackerone team as quickly as possible, especially because I know it could take some days for a response. then I thought about sending a message to some of the Hackerone triage team, I was expecting a response within a few hours but it doesn't happened. So I decided to send a message to @jobert, and I was really surprised by his quick and kind response.


+ ## How the Hackerone team was responsible for fixing this issue and how they responded:
> I was really surprised by Hackerone quick response, they have fixed the issue and let me retest it in less than 24 hours. They are a very collaborative team.

+ ## Supporting Material/References:
> + [What to Do If You Inadvertently Expose an AWS Access Key.](https://aws.amazon.com/blogs/security/what-to-do-if-you-inadvertently-expose-an-aws-access-key/)
> + [How to prevent server-side request forgery. (SSRF)](https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html)
> + [How to prevent HTML injection.](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)
> + [How to write high quality reports.](https://docs.hackerone.com/en/articles/8475116-quality-reports)
> + [How To: Server-Side Request Forgery (SSRF)](https://www.hackerone.com/application-security/how-server-side-request-forgery-ssrf)


Thanks for reading and have a nice day!

*Best Regards,*
*@Mega7*

## Timeline (public activity, condensed from the rendered page)

- 2023-11-23 12:52 UTC: megaman_ submitted the report (PoC redacted in public version).
- 2023-11-24: HackerOne triage (h1_analyst_pranav) acknowledged, validated, moved to Pending program review; HackerOne staff (0daybookworm) set Triaged, started Retesting (retest bounty $50) after a fix was deployed within about a day.
- megaman_ comment: messaged HackerOne triage analysts directly and then the CEO to flag urgency; later suggested an "Urgent Security Reports" option.
- 2023-11-24: megaman_ retest result: "No, the fix works" (HTML no longer reflected).
- 2023-11-25 megaman_ impact comment (severity discussion): SSRF reached 169.254.169.254; obtained AWS AccessKeyId and SecretAccessKey; potential to manipulate AWS resources (EC2, S3), data loss, account takeover via stored XSS from malicious files, and arbitrary command execution (for example via `aws ssm send-command` with `AWS-RunShellScript`).
- 2023-11-28: HackerOne staff raised severity from high (8.8) to critical (9.8); report resolved; $25,000 bounty awarded.
- 2023-12-08: disclosure agreed; severity updated to critical (10.0).
