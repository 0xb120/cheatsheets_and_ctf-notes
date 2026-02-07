---
raindrop_id: 1495845394
raindrop_highlights:
  695ea9b18a65bffb612d2566: 183522dee94f90d3c7091ab883bb4dc6
  695ea9ba091512db7f15eb7f: 05018a886cb5767b706b128111c6824d
  695eaa9e00a3e3712b468217: 023105e92eacd563c52c120d669b4091
  695eac0e091512db7f16714a: 95ecc56efe941df46ce6dcff038f2784
  695eac14091512db7f1672a4: a44271dc34901c9352111b8dd36a2585
  695eac353264aee877450b3f: cd4db4c5876345d593859ed58d8679d9
  695eac444b4a97ea1f7809bd: 5bf2148409f7341ce19a70df46a1de8d
  695eac4b16d39ad8c086ea77: 87cf1c4b84fdfe48a13a071c580c1f46
  695eac61a060de90ead29150: e11e5deeea4a9a77396a0b745eae8bb3
  695eac6912eae02cb3b5bcfa: 0e9b3deaaf393085f1ea9e62c2f9b859
  695eac71143ff1c0f9e4df15: 694d65113324a1f74f68d97d88545725
  695eac89143ff1c0f9e4e48a: a140a7524b9c61007560f7ad99503279
  695eaca2091512db7f169475: 8d9248955c8230903c34599167b00847
  695eaca5143ff1c0f9e4eac7: 1cd5a0c327b8eace285bcb4f1b2687ae
  695eacc9091512db7f169dad: 84e8bb34ac5c5b332b74bcb978a9fe98
  695eaccf5e3fc20682d73f2b: 60ef133b74dcb464872cc354bc848281
  695eacee4e820e2e2eef1bf8: 120abc55d79068649a3cd37d973cf42b
  695ead1816d39ad8c08717d9: 77c9e288998e0b952686e09be7fd9e4d
  695ead21091512db7f16af9f: 6bc6f622c8f4e4fa283bd160e165ebbc
  695ead2800a3e3712b471187: bcc9ee0c675be3e3f9c1ce21995b16bb
  695ead39d0dd7d84c93f6c9d: d710c70ab746244b36061285b2278097
  695ead4716d39ad8c08720b3: 8e12a06c64ce1b7ee7295e03a10c8865
  695ead6b3264aee877454e77: ed2d24cf6a0934ad815367dba24353ae
  695ead7c4b4a97ea1f784c69: fd6342945b63b332e5ed66a18a487657
  695ead914b4a97ea1f784ff5: 8970c518b5d50829f5b55e9615c436f5
  695ead9f143ff1c0f9e51ccb: d1c824447ddfb5ef296bc2a208b23c20
  695eadab587e1384d5ef43d2: fc13657320110817bf1a89ba50da1231
  695eadb64e820e2e2eef40bf: 0d4aea3cb770f36b549f2a9f1f6d8f3c
title: "InstaVM - Secure Execution of AI Generated Code"

description: |-
  Execute code securely in isolated virtual machines with our high-performance cloud infrastructure.

source: https://instavm.io/blog/analysed-4000-to-create-security-agent-cli

created: 2025-12-19
sync-date: 1767878124251
tags:
  - "_index"

 
  - "tech-blog" 
  - "AI" 
  - "LLM" 
  - "DAST"

---
# InstaVM - Secure Execution of AI Generated Code

![](https://instavm.io/blog-images/bruteforce-pdf.png)

> [!summary]
> Execute code securely in isolated virtual machines with our high-performance cloud infrastructure.





I've been using Gemini CLI, Claude Code and similar agents a lot lately.
After seeing it crack a PDF in seconds for $0.27, being a security researcher, my next question was obvious: Can I teach it to hack APIs?
I built a mitmproxy AI agent using 4000 paid security disclosures
A Better Approach
was to not log everything from mitmdump to the stdout
So, I logged everything in a log.txt file and asked CLI tool to use Regex, Grep etc. in the file for things it is looking for.
I also decided to convert "starting mitmdump and directing its log to a file" into a command (a feature available in both Gemini CLI and Claude Code) - /start-mitm
~/Work/mitmclaude$ cat .claude/commands/start-mitm.md # Start mitmproxy capture Start mitmproxy to capture HTTP/HTTPS traffic with full request and response bodies. ## Instructions Run the following command to start mitmdump in the background: mitmdump -w traffic.mitm --set flow_detail=3 2>&1 | tee log.txt &
Naive Approach
Though these LLMs have some capability to do security checks - I thought it will surely benefit from some additional real world examples of security issues right here in its context. So, I put the description and reproducibility steps of security bugs I had found in my career right here in their system instructions (via files such as GEMINI.md or CLAUDE.md)
This makes the system instructions a little too big for these LLMs' comfort with the description of around 150 security issues and they start to complain.
Borrowing an idea from command of /start-mitm above, I thought why not put descriptions and how to find that one category of bugs into one command, say /check-for-idor will refer to markdown file CHECK-FOR-IDOR.md which has only IDOR related vulnerabilities details.
And lets not dump all the descriptions of the bug report as it is - the direct dump of them from hf - but rather only the learnings from them and how to look for them. I used one of the top LLMs to make this conversion from security reports to How to look for these issues .
From 150 security reports to 4000
With the architecture finalized it was time to improve the scope of the tool. My own security bugs were a good starting point but I wanted a world class security researcher. So I went to the HackerOne's public disclosures. There is one dump available at hugging face https://huggingface.co/datasets/Hacker0x01/hackerone_disclosed_reports?viewer_api=true

There are roughly 10000 bugs in there and I filtered out 4000 from them which had some bounty payment, a proxy for a good impact bug, I assumed.
# Logic: Only train on bugs that paid out df = df[ (df['bounty_amount'] > 0) & (df['vulnerability_type'].isin(['IDOR', 'SSRF', 'RCE'])) ]
With the data from 4000 security issues, all the command's markdown files were modified to include the new learning and new ways to find the issues
Transition From Researcher Controlled Tools To An Agentic Tool
I wanted the LLM to figure out what are the available commands and invoke them on their own
achieve this in three ways
create MCP endpoints which search these "commands" as an endpoint individually. So, when MCP calls /list-tools it will see those commands as the individual tools.
or convert them into Skills if you are using Claude Code
or convert them into Skills and use coderunner to help bridge the gap for Skill->MCP conversion for you.
The content of a Skill is exactly same as the command. The same markdown file can be just used as a Skill by putting it appropriate folder.
A sample skill of finding IDOR
# ~/Work/mitmclaude$ cat .claude/skills/mitmfindidor.md

---
description: Find IDOR (Insecure Direct Object Reference) vulnerabilities in captured traffic. Use when user asks about authorization issues, sequential IDs, or accessing other users' data.
---

# Find IDOR Vulnerabilities

Analyze the mitmproxy dump (log.txt) for IDOR vulnerabilities for: $ARGUMENTS

## High-Value IDOR Patterns (from 132 real HackerOne bounty reports)

### 1. User/Account Object References
```
user_id, userId, user-id, uid, account_id, accountId
customer_id, customerId, member_id, memberId
profile_id, owner_id, creator_id, author_id
```
**Real example**: `https://zomato.com/gold/payment-success?subscription_id=XXX&user_id=YYY`

### 2. Resource Object References
```
order_id, orderId, booking_id, bookingId, reservation_id
transaction_id, txn_id, payment_id, invoice_id
document_id, doc_id, file_id, attachment_id
report_id, ticket_id, case_id, issue_id
```
**Real example**: `/api/shopify/orders/{order_id}` - change order_id to access other orders

### 3. Organizational Object References
```
project_id, projectId, team_id, teamId, group_id, groupId
workspace_id, org_id, organization_id, company_id
board_id, channel_id, room_id, space_id
```
**Real example**: `PUT /boards/{board_id}.json` - GitLab private project label access

### 4. Content Object References
```
media_code, media_id, image_id, video_id, asset_id
post_id, postId, comment_id, message_id, thread_id
article_id, content_id, item_id, entry_id
```
**Real example**: `media_code=2013124` - sequential IDs expose other users' media

### 5. Session/Token References (High Impact)
```
session_id, sessionId, subscription_id, subscriptionId
card_id, cardId, fuel_card_id, membership_id
api_key_id, token_id, credential_id
```
**Real example**: `activateFuelCard?id=XXX` - Uber driver UUID enumeration

## ID Encoding Patterns to Decode

| Pattern | Example | Decode Method |
|---------|---------|---------------|
| Base64 numeric | `MTIzNDU2` | `echo MTIzNDU2 \| base64 -d` → 123456 |
| Hex | `0x1E240` | Convert to decimal → 123456 |
| UUID v1 | Contains timestamp | Extract timestamp component |
| Short hash | `a1b2c3` | May be truncated MD5 of sequential |
| Padded | `000012345` | Strip padding, increment |

## Where to Find IDORs

### URL Path Parameters (Most Common)
```
/api/v1/users/{id}/profile
/api/v1/orders/{id}/details
/api/v1/documents/{id}/download
/campaign-manager-api/accounts/{id}
```

### Query Parameters
```
?user_id=12345&action=view
?subscription_id=XXX&user_id=YYY
?media_code=2013124
```

### Request Body (JSON/Form)
```json
{"user_id": 12345, "action": "delete"}
{"board": {"id": 857058, "labels": [{"id": 123}]}}
```

### Headers (Rare but High Impact)
```
X-User-Id: 12345
X-Account-Id: 67890
```

## Severity Rating

| Access Type | Severity | Example |
|-------------|----------|---------|
| Read other users' PII | **CRITICAL** | View email, phone, address |
| Modify other users' data | **HIGH** | Edit profile, delete content |
| Access other users' orders/transactions | **HIGH** | View order history, payment info |
| Read other users' private content | **MEDIUM** | View private posts, documents |
| Enumerate user existence | **LOW** | Confirm if user_id exists |
| Access public-ish data | **INFO** | View subscription dates |

## Testing Methodology

### Step 1: Identify Candidate Parameters
Search for ID patterns in traffic:
```bash
grep -iE '(user|account|order|session|subscription|member|card|document|file|project|team|group)[-_]?id' log.txt
```

### Step 2: Check for Sequential/Predictable IDs
```bash
# Extract numeric IDs and check if sequential
grep -oE 'id[=:]["'\'']?[0-9]+' log.txt | sort -u
```

### Step 3: Test Authorization
```bash
# Test with ID ± 1
curl -H "Cookie: victim_session" "https://target.com/api/resource/12345"
curl -H "Cookie: victim_session" "https://target.com/api/resource/12344"  # Another user's
```

### Step 4: Verify Impact
- Does response contain different user's data?
- Can you perform actions (edit/delete) on other user's resources?
- What sensitive fields are exposed?

## Output Format

For each finding report:

```
## IDOR Finding: [Brief Description]

**Endpoint**: `METHOD https://target.com/path`
**Parameter**: `param_name` in [path|query|body]
**ID Type**: [Sequential|Base64|UUID|Hash]
**Current Value**: `12345`
**Severity**: [CRITICAL|HIGH|MEDIUM|LOW]

**Evidence**:
[Show request/response snippets]

**Impact**:
- What data is exposed
- What actions can be performed

**Test Command**:
curl -X METHOD 'https://target.com/...' -H 'Cookie: ...'

**Remediation**:
- Implement proper authorization checks
- Use indirect references (mapping table)
- Validate user owns the resource
```

## False Positives to Ignore

- Analytics/tracking endpoints (write-only, no data returned)
- Public content IDs (movie IDs, product catalog)
- Resource IDs that return same data regardless of auth
- IDs that require valid session
~/Work/mitmclaude$ tree .claude/skills .claude/skills ├── mitm-find-auth.md ├── mitm-find-bizlogic.md ├── mitm-find-callback.md ├── mitm-find-checksum.md ├── mitm-find-enumerable.md ├── mitm-find-idor.md ├── mitm-find-insecure.md ├── mitm-find-otp.md ├── mitm-find-pii.md ├── mitm-find-referer.md ├── mitm-find-secrets.md ├── mitm-find-sqli.md ├── mitm-find-ssrf.md ├── mitm-list-apis.md ├── mitm-report.md ├── mitm-security-audit.md └── mitm-subdomains.md 1 directory, 17 files
After that we can just either let the CLI tools decide what Skill to use or ask it to use a particular set of skill(s) in plain English as following -

Find security issues in example.com

or

Check for `idor` and `auth` issues in example.com
This blog is meant to provide a way to club mitmproxy with an exiting LLM CLI tool and make it more intelligent with existing bug bounty reports.
Resources
security-skills - Security testing skills derived from 4000+ HackerOne bounty reports
HackerOne Disclosed Reports Dataset - 10,000+ public bug bounty reports