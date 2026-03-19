# 7 Free Bash/Python Scripts to Automate Your Sales Process

Sales automation doesn't have to be expensive. Here are 7 completely free scripts you can start using today to automate repetitive sales tasks.

## 1. Lead Enrichment Script (Bash + curl)
Enrich lead data with company information from public APIs.

```bash
#!/bin/bash
# enrich_lead.sh - Takes a domain and returns company info
DOMAIN=$1
if [ -z "$DOMAIN" ]; then
  echo "Usage: $0 <domain.com>"
  exit 1
fi

# Using Clearbit's free API (limited) or similar
# Replace this URL with your actual API endpoint (Clearbit, Hunter.io, etc.)
# For demo, we'll use a free company info API
response=$(curl -s "https://api.company-info.com/v3/domain/$DOMAIN")
echo "$response" | jq .
```

## 2. Email Verification Tool (Python)
Verify if an email address is valid without sending an email.

```python
#!/usr/bin/env python3
import smtplib
import re

def verify_email(email):
    # Basic syntax check
    pattern = r'^[\w\.-]+@[\w\.-]+\.\w+$'
    if not re.match(pattern, email):
        return False, "Invalid email format"
    
    domain = email.split('@')[1]
    
    # Check MX records
    import dns.resolver
    try:
        mx_records = dns.resolver.resolve(domain, 'MX')
        mx_record = str(mx_records[0].exchange)
        
        # Connect to SMTP server
        server = smtplib.SMTP(timeout=10)
        server.connect(mx_record)
        server.helo()
        # Replace me@domain.com with your actual sender address
        server.mail('me@domain.com')
        code, message = server.rcpt(str(email))
        server.quit()
        
        if code == 250:
            return True, "Email is valid"
        else:
            return False, f"SMTP error: {code} {message}"
    except Exception as e:
        return False, f"DNS/SMTP error: {e}"

if __name__ == "__main__":
    email = input("Email to verify: ")
    is_valid, msg = verify_email(email)
    print(f"Valid: {is_valid}")
    print(f"Reason: {msg}")
```

## 3. Meeting Scheduler (Bash + curl)
Find mutual availability without Calendly.

```bash
#!/bin/bash
# schedule_meeting.sh - Simple time slot proposer
# In a real implementation, this would integrate with Google Calendar API
# For now, a template for generating time slots

echo "Proposing 3 time slots for next week:"
for i in {1..3}; do
  # Generate random future dates/times (simplified)
  day=$((RANDOM % 5 + 1))  # Monday-Friday
  hour=$((RANDOM % 8 + 9))  # 9am-5pm
  minute=$((RANDOM % 2 * 30))  # 0 or 30
  printf "Option %d: %02d:%02d\n" $i $hour $minute
done
```

## 4. Follow-up Automation (Python)
Automate polite follow-up emails based on engagement.

```python
#!/usr/bin/env python3
import smtplib
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart
import time

def send_followup(to_email, template_name, delay_days=3):
    """
    Send a follow-up email after a delay.
    In production, this would be triggered by lack of reply.
    """
    # This is a simplified version - in reality you'd track sent emails
    # and use a scheduler like cron or a background job
    
    templates = {
        "gentle_checkin": "Just checking in to see if you had a chance to review my last email?",
        "value_add": "I came across this article that might be relevant to our discussion: [link]",  # Replace [link] with your actual article URL
        "breakup": "I'll assume you're busy for now and will reach out again next quarter."
    }
    
    body = templates.get(template_name, "Following up on my previous message.")
    
    # Actual sending would go here
    print(f"Would send to {to_email}: {body}")
    print(f"Scheduled for {delay_days} days from now")

if __name__ == "__main__":
    send_followup("prospect@example.com", "value_add")
```

## 5. Pipeline Health Check (Bash + jq)
Quick assessment of your sales pipeline health.

```bash
#!/bin/bash
# pipeline_health.sh - Analyze pipeline data (CSV format)
# Expected CSV: stage,amount,probability,expected_close_date

if [ ! -f "pipeline.csv" ]; then
  echo "Please provide pipeline.csv with columns: stage,amount,probability,expected_close_date"
  exit 1
fi

echo "=== SALES PIPELINE HEALTH CHECK ==="
echo ""

# Weighted pipeline value
weighted_value=$(awk -F, '{sum+=$2*$3/100} END {printf "%.2f", sum}' pipeline.csv)
echo "Weighted Pipeline Value: \$$weighted_value"

# Stage distribution
echo ""
echo "Stage Distribution:"
awk -F, '{stages[$1]+=$2} END {for (s in stages) printf "%s: \$$%.2f\n", s, stages[s]}' pipeline.csv

# Close rate by stage (simplified)
echo ""
echo "Recommendation: Focus on stages with highest conversion rates"

```

## 6. Call Analysis Helper (Python)
Transcribe and analyze sales calls (basic version).

```python
#!/usr/bin/env python3
# call_analyzer.py - Basic call transcription and keyword spotting

import re

def analyze_call_transcript(transcript):
    """Analyze a sales call transcript for key indicators"""
    
    # Positive signals
    positive_keywords = [
        'budget', 'timeline', 'implementation', 'team', 'stakeholder',
        'roi', 'cost', 'pricing', 'contract', 'start date'
    ]
    
    # Negative signals
    negative_keywords = [
        'not interested', 'too expensive', 'not now', 'call me back',
        'send me information', 'maybe later'
    ]
    
    transcript_lower = transcript.lower()
    
    positive_count = sum(1 for keyword in positive_keywords if keyword in transcript_lower)
    negative_count = sum(1 for keyword in negative_keywords if keyword in transcript_lower)
    
    score = positive_count - negative_count
    
    print(f"Positive signals: {positive_count}")
    print(f"Negative signals: {negative_count}")
    print(f"Net score: {score}")
    
    if score > 2:
        return "Hot lead - strong buying signals"
    elif score > 0:
        return "Warm lead - some interest"
    else:
        return "Cold lead - needs nurturing"

# Example usage
if __name__ == "__main__":
    sample_transcript = """
    Hi, I'm looking for a solution to help with our sales tracking.
    We have a budget of $10k and want to implement by Q3.
    Can you show me pricing and implementation timeline?
    """
    print(analyze_call_transcript(sample_transcript))
```

## 7. Proposal Generator (Bash + templating)
Create customized proposals quickly.

```bash
#!/bin/bash
# generate_proposal.sh - Create a sales proposal from template

TEMPLATE_DIR="./proposal_templates"
OUTPUT_DIR="./proposals"

if [ ! -d "$TEMPLATE_DIR" ]; then
  echo "Please create proposal_templates directory with your templates"
  exit 1
fi

# Simple variable substitution
CLIENT_NAME=$1
PROJECT_SCOPE=$2
PROJECT_VALUE=$3

if [ -z "$CLIENT_NAME" ] || [ -z "$PROJECT_SCOPE" ] || [ -z "$PROJECT_VALUE" ]; then
  echo "Usage: $0 <client_name> <project_scope> <project_value>"
  exit 1
fi

# Create output directory if it doesn't exist
mkdir -p "$OUTPUT_DIR"
# Use a simple template (in practice, use a real templating engine)
cat > "$OUTPUT_DIR/${CLIENT_NAME}_proposal.txt" << EOF
Proposal for: $CLIENT_NAME
Date: $(date +%Y-%m-%d)

Project Scope:
$PROJECT_SCOPE

Investment: \$$PROJECT_VALUE

Next Steps:
1. Sign and return this proposal
2. We'll schedule a kickoff call
3. Delivery timeline: 4-6 weeks

Thank you for the opportunity!
EOF

echo "Proposal generated: $OUTPUT_DIR/${CLIENT_NAME}_proposal.txt"
```

## How to Use These Scripts

1. **Save each script** to a file with the appropriate extension (.sh for bash, .py for python)
2. **Make them executable**: `chmod +x script_name.sh` or `chmod +x script_name.py`
3. **Install dependencies**: 
   - For Python scripts: `pip install dnspython` (for email verification)
   - For bash scripts: most tools come standard on macOS/Linux
4. **Customize** them for your specific sales process
5. **Combine and automate** - use cron or background jobs for recurring tasks

## Why This Approach Works

- **Zero cost**: All scripts use free, public APIs or standard tools
- **Immediate value**: Start saving time on repetitive tasks today
- **Learn by doing**: Modify and improve the scripts to fit your exact needs
- **Scalable**: As you grow, you can replace free APIs with paid ones as needed
- **Transferable skills**: You're learning automation, scripting, and API integration

## Next Steps

1. Pick ONE script that solves your most painful repetitive task
2. Implement it this week
3. Measure the time saved
4. Move to the next script

Remember: The goal isn't to build the perfect automation system overnight. It's to start small, prove the concept, and gradually eliminate the repetitive tasks that eat up your selling time.

**Free bonus**: Want a ready-to-use version of these scripts? Check out the companion repository at [github.com/reggie-rex/sales-automation-scripts](https://github.com/reggie-rex/sales-automation-scripts) (example link - replace with actual if you create it)

---

*Created by Reginald "Reggie" Rex - AI Systems Engineering Specialist*
*For more sales engineering resources, visit: https://shawnfixer.gumroad.com/l/ruxqr*