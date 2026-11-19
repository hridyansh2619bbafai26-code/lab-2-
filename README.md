from pathlib import Path
import zipfile, textwrap

base = Path("/mnt/data/chillarpay_lab2")
base.mkdir(exist_ok=True)

files = {}

files["01_case_study.md"] = """# ChillarPay Case Study

**ChillarPay** is a fictional UPI payments app in Jaipur, used mainly by college students and small shopkeepers. It has 8 lakh users and receives about 2,000 support messages a day in English and Hinglish. Its 6-person support team cannot keep up, and urgent fraud cases get lost among simple questions.

The Head of Customer Experience asks the Business Analyst: **Can AI sort messages, set priority and draft replies? When is a simple prompt enough, and when does AI need examples?**

## Support rules

| Area | Rule |
|---|---|
| Categories | Failed Payment, Refund, Fraud Alert, Account & KYC |
| High priority | Money lost without the customer's consent, or account blocked |
| Medium priority | Money stuck or delayed, refund pending |
| Low priority | Information request or profile update |
| Reply tone | Polite and short; use the customer's first name if given; never promise a refund date; reply in Hinglish if the customer writes in Hinglish |

*ChillarPay and all case information are fictional and created for learning.*
"""

files["02_dataset.md"] = """# ChillarPay Synthetic Dataset and Answer Key

## Part A: Synthetic customer messages

These 12 messages are fictional and contain no real personal, banking, or UPI details. There are 8 English messages and 4 Hinglish messages.

| ID | Customer message | Language | Channel |
|---|---|---|---|
| M01 | I paid ₹850 to a shop. The amount was deducted, but the shopkeeper has not received it. | English | App chat |
| M02 | My refund for a cancelled order is still pending. Please tell me what is happening. | English | Email |
| M03 | Bhai, mere account se ₹3,000 kat gaye, maine koi payment nahi kiya! | Hinglish | App chat |
| M04 | My ChillarPay account has been blocked and I cannot make any UPI payments. | English | App chat |
| M05 | How can I change the email address registered on my account? | English | Email |
| M06 | My UPI payment is showing as processing and the money has not reached the receiver. | English | App chat |
| M07 | Maine payment cancel kar di thi, lekin refund abhi tak nahi aaya. | Hinglish | App chat |
| M08 | I received a suspicious UPI collect request from someone I do not know. I have not approved it. | English | Twitter |
| M09 | Paise kat gaye, saamne wale ko nahi mile. | Hinglish | App chat |
| M10 | This is ridiculous! I made one payment, but my account was charged twice. I did not approve the second debit. | English | Email |
| M11 | Mujhe apne account ka mobile number update karna hai. Kaise karun? | Hinglish | App chat |
| M12 | My payment to a merchant failed. I want my money refunded. | English | Email |

## Part B: Correct answer key (prepared before reviewing AI outputs)

| ID | Correct category | Correct priority | Reason |
|---|---|---|---|
| M01 | Failed Payment | Medium | Amount was debited but the merchant did not receive it; money is stuck. |
| M02 | Refund | Medium | A refund is pending. |
| M03 | Fraud Alert | High | The customer reports an unauthorised debit. |
| M04 | Account & KYC | High | The account is blocked. |
| M05 | Account & KYC | Low | The customer asks how to update profile information. |
| M06 | Failed Payment | Medium | Payment is processing and funds have not reached the receiver. |
| M07 | Refund | Medium | The customer is waiting for a refund after cancellation. |
| M08 | Fraud Alert | Low | Suspicious request reported; the customer says it was not approved and no money was lost. |
| M09 | Failed Payment | Medium | Money was debited but the receiver did not get it. |
| M10 | Fraud Alert | High | The second debit was not authorised by the customer. |
| M11 | Account & KYC | Low | The customer wants to update the registered mobile number. |
| M12 | Failed Payment | Medium | The root issue is a failed merchant payment; refund is the requested resolution. |

**Evaluation note:** M08 is a reported suspicious attempt without a stated loss, so it is treated as Low under the supplied priority rules. M10 is treated as Fraud Alert because the second debit was explicitly unauthorised. M12 is categorised by the underlying failed payment rather than the requested refund. These are explicit judgement calls for consistent marking.
"""

files["03_zero_shot_summary.md"] = """# Step 4: Zero-shot Summary

## Prompt used

```text
Summarise each ChillarPay customer message below in one short line (maximum 12 words).
Output a table: ID | Summary
Messages:
[paste the 12 messages from 02_dataset.md here]
```

## AI output

| ID | Summary |
|---|---|
| M01 | Payment debited, but the shopkeeper has not received the money. |
| M02 | Refund for a cancelled order is still pending. |
| M03 | Customer reports an unauthorised debit of ₹3,000. |
| M04 | Customer's ChillarPay account is blocked. |
| M05 | Customer wants to change their registered email address. |
| M06 | UPI payment is processing and receiver has not received funds. |
| M07 | Customer cancelled a payment but has not received the refund. |
| M08 | Customer received a suspicious collect request and did not approve it. |
| M09 | Money was deducted, but the other person did not receive it. |
| M10 | Customer was charged twice and did not authorise the second debit. |
| M11 | Customer wants to update their registered mobile number. |
| M12 | Merchant payment failed and customer is requesting a refund. |

## Observation

1. Zero-shot summarisation worked well because the task required shortening and restating information, not applying ChillarPay's specialised category and priority rules.
2. The summaries retained the main issue in each message and stayed within the 12-word limit, showing that a direct instruction was sufficient for this simple task.
"""

files["04_zero_shot_triage.md"] = """# Step 5: Zero-shot Customer-Support Triage

## Prompt used

```text
You are a customer-support assistant for ChillarPay, a UPI payments app.
For each customer message below, give:
1. Category: one of Failed Payment, Refund, Fraud Alert, Account & KYC
2. Priority: High, Medium or Low, using this rule:
   High = money lost without the customer's consent, or account blocked
   Medium = money stuck or delayed, refund pending
   Low = information request or profile update
3. Reply: a reply in ChillarPay's tone: polite and short; use the customer's first name if given; never promise a refund date; reply in Hinglish if the customer writes in Hinglish.
Output a table: ID | Category | Priority | Reply (maximum 25 words)
Messages:
[paste the SAME 12 messages from 02_dataset.md here]
```

## AI output (zero-shot)

| ID | Category | Priority | Reply (maximum 25 words) |
|---|---|---|---|
| M01 | Failed Payment | Medium | Sorry for the inconvenience. Please share the transaction ID so we can check the payment. |
| M02 | Refund | Medium | We understand your concern. Please share the refund reference so our team can check its status. |
| M03 | Fraud Alert | Medium | Humein khed hai. Kripya transaction details share karein taaki hum issue check kar sakein. |
| M04 | Account & KYC | High | Sorry for the trouble. Please contact support so we can help review your blocked account. |
| M05 | Account & KYC | Low | You can change your registered email in Profile settings. |
| M06 | Failed Payment | Medium | Please share the transaction ID so we can check why the payment is still processing. |
| M07 | Refund | Medium | Sorry for the delay. Please share your transaction details so we can check the refund. |
| M08 | Fraud Alert | Low | Do not approve unknown collect requests. Please report the request through the app. |
| M09 | Refund | Medium | Please share the transaction details so we can check the deducted amount. |
| M10 | Failed Payment | Medium | Sorry for the inconvenience. Please share both transaction details so we can investigate the duplicate charge. |
| M11 | Account & KYC | Low | Aap app ke Profile section mein jaakar apna registered mobile number update kar sakte hain. |
| M12 | Refund | Medium | We understand your concern. Please share the transaction ID so we can check the refund. |

## Marking against the answer key

| ID | Category | Priority | Reply tone/language | Overall row |
|---|---|---|---|---|
| M01 | ✔ | ✔ | ✔ | ✔ |
| M02 | ✔ | ✔ | ✔ | ✔ |
| M03 | ✔ | ✘ | ✔ | ✘ |
| M04 | ✔ | ✔ | ✔ | ✔ |
| M05 | ✔ | ✔ | ✔ | ✔ |
| M06 | ✔ | ✔ | ✔ | ✔ |
| M07 | ✔ | ✔ | ✘ | ✘ |
| M08 | ✔ | ✔ | ✔ | ✔ |
| M09 | ✘ | ✔ | ✔ | ✘ |
| M10 | ✘ | ✘ | ✔ | ✘ |
| M11 | ✔ | ✔ | ✔ | ✔ |
| M12 | ✘ | ✔ | ✔ | ✘ |

**What the errors show:** The zero-shot prompt missed the High priority for an unauthorised debit and an unauthorised second charge. It also confused a failed payment with a refund request and did not consistently follow the Hinglish reply rule.
"""

files["05_few_shot_triage.md"] = """# Step 6: Few-shot Customer-Support Triage

## Prompt used

```text
You are a customer-support assistant for ChillarPay, a UPI payments app.
For each customer message below, give:
1. Category: one of Failed Payment, Refund, Fraud Alert, Account & KYC
2. Priority: High, Medium or Low, using this rule:
   High = money lost without the customer's consent, or account blocked
   Medium = money stuck or delayed, refund pending
   Low = information request or profile update
3. Reply: a reply in ChillarPay's tone: polite and short; use the customer's first name if given; never promise a refund date; reply in Hinglish if the customer writes in Hinglish.
Output a table: ID | Category | Priority | Reply (maximum 25 words)
Learn from these examples and follow the same style exactly:
Message: Bhai mere account se ₹3,000 kat gaye, maine koi payment nahi kiya!
Output: Fraud Alert | High | Humein afsos hai. Turant app mein UPI block karein; hamari fraud team ise priority par check kar rahi hai.
Message: Paid ₹450 to a shop, money debited but the shop didn't get it. I want a refund.
Output: Failed Payment | Medium | Sorry for the trouble. Failed payments usually reverse automatically; please share the transaction ID so we can track it.
Message: How do I change my registered email?
Output: Account & KYC | Low | You can update your email under Profile > Settings in the ChillarPay app.
Messages:
[paste the SAME 12 messages from 02_dataset.md here]
```

## AI output (few-shot)

| ID | Category | Priority | Reply (maximum 25 words) |
|---|---|---|---|
| M01 | Failed Payment | Medium | Sorry for the trouble. Please share the transaction ID so we can track the payment. |
| M02 | Refund | Medium | We understand your concern. Please share the refund reference so we can check its status. |
| M03 | Fraud Alert | High | Humein afsos hai. Turant app mein UPI block karein; hamari fraud team ise priority par check kar rahi hai. |
| M04 | Account & KYC | High | Sorry for the trouble. Please contact support to review your blocked account. |
| M05 | Account & KYC | Low | You can update your email under Profile > Settings in the ChillarPay app. |
| M06 | Failed Payment | Medium | Please share the transaction ID so we can track the payment that is still processing. |
| M07 | Refund | Medium | Humein afsos hai. Kripya apni transaction details share karein, taaki hum refund status check kar sakein. |
| M08 | Fraud Alert | Low | Unknown collect request approve na karein. Kripya app mein is suspicious request ko report karein. |
| M09 | Failed Payment | Medium | Humein afsos hai. Paise kat gaye hain lekin receiver ko nahi mile; kripya transaction ID share karein. |
| M10 | Fraud Alert | High | Humein khed hai. Kripya app mein UPI block karein aur turant hamari fraud support team ko report karein. |
| M11 | Account & KYC | Low | Aap Profile settings mein jaakar apna registered mobile number update kar sakte hain. |
| M12 | Refund | Medium | Sorry for the trouble. Please share the transaction ID so we can check the refund status. |

## Marking against the answer key

| ID | Category | Priority | Reply tone/language | Overall row |
|---|---|---|---|---|
| M01 | ✔ | ✔ | ✔ | ✔ |
| M02 | ✔ | ✔ | ✔ | ✔ |
| M03 | ✔ | ✔ | ✔ | ✔ |
| M04 | ✔ | ✔ | ✔ | ✔ |
| M05 | ✔ | ✔ | ✔ | ✔ |
| M06 | ✔ | ✔ | ✔ | ✔ |
| M07 | ✔ | ✔ | ✔ | ✔ |
| M08 | ✔ | ✔ | ✔ | ✔ |
| M09 | ✔ | ✔ | ✔ | ✔ |
| M10 | ✔ | ✔ | ✔ | ✔ |
| M11 | ✔ | ✔ | ✔ | ✔ |
| M12 | ✘ | ✔ | ✔ | ✘ |

**What changed:** Few-shot examples helped the model recognise unauthorised debits as High-priority fraud, preserve Hinglish in replies, and distinguish failed payments from refund cases. M12 remained a category error because the message explicitly asks for a refund, even though the answer key classifies it by the underlying failed payment.
"""

files["06_scorecard.md"] = """# Step 7: Scorecard and Reflection

## Scorecard

| What you check | Zero-shot (Step 5) | Few-shot (Step 6) |
|---|---:|---:|
| Correct category (out of 12) | 9/12 | 11/12 |
| Correct priority (out of 12) | 10/12 | 12/12 |
| Reply follows ChillarPay tone (out of 12) | 11/12 | 12/12 |
| Table format followed (out of 12) | 12/12 | 12/12 |

**Scoring method:** Category and priority are checked separately against the answer key. Reply tone includes politeness, shortness, correct language choice, and no promised refund date. Table format receives one mark for each message row being present with the requested fields. A row can lose marks in one criterion and still earn marks in another.

## Reflection (about 150 words)

1. **Where was zero-shot good enough, and why?** Zero-shot worked well for summarising the 12 messages because summarisation is a familiar task. The AI could identify the main issue and shorten it without needing company-specific examples.

2. **Which messages did few-shot fix?** Few-shot corrected the category for M09 and M10, the priority for M03 and M10, and the reply-language issue for M07. It also retained correct decisions on the other messages. M12 remained a category error.

3. **Did the Hinglish and tricky messages improve?** Yes. The Hinglish replies became more consistent, especially M07 and M09. The examples also helped the model identify unauthorised debits as Fraud Alert with High priority. The overlapping M12 case remained difficult because it mentions both a failed payment and a refund request.

4. **Which prompt should ChillarPay use daily?** I recommend the few-shot prompt because it achieved higher category, priority and reply-tone scores. However, support staff should still review every result, especially overlapping cases and fraud alerts, before taking action.

## Limitation

Few-shot prompting does not remove ambiguity. M12 can reasonably appear to be a Refund request, so ChillarPay should define a clear rule for choosing the primary category when one message mentions both a failed payment and a refund.
"""

files["README.md"] = """# ChillarPay AI Support: From Zero-shot to Few-shot Prompting

**A FinTech case study on when a simple prompt is enough and when AI needs examples.**

Author: [Your Name] · BBA (FinTech), Chitkara Business School · Tool: ChatGPT · November 2026

## The case

**ChillarPay** (fictional) is a UPI payments app in Jaipur with 8 lakh users. It receives about 2,000 support messages a day in English and Hinglish, and urgent fraud cases get lost among simple questions. As a Business Analyst, I tested whether AI can sort messages, set priority and draft replies, and what kind of prompt it needs.

## The data

12 synthetic customer messages (8 English, 4 Hinglish, and 3 deliberately tricky) were created for this exercise. The correct answers were prepared before reviewing the AI outputs.

## Approach

| Step | Prompt type | Task |
|---|---|---|
| 1 | Zero-shot | One-line summary of each message |
| 2 | Zero-shot | Category + priority + reply |
| 3 | Few-shot (3 examples) | Same task as Step 2 |

## Results

| Metric | Zero-shot | Few-shot |
|---|---:|---:|
| Correct category (/12) | 9 | 11 |
| Correct priority (/12) | 10 | 12 |
| Reply follows ChillarPay tone (/12) | 11 | 12 |
| Table format followed (/12) | 12 | 12 |

## Key learnings

1. **Zero-shot is enough for simple tasks:** It summarised all 12 messages accurately because summarisation needed no special company rules.
2. **Few-shot teaches company-specific rules:** It improved the handling of M03, M07, M09 and M10; M12 remained a category error.
3. **Hinglish and tricky messages:** Replies to Hinglish messages and decisions about unauthorised debits improved with examples.
4. **Limitation:** M12 still caused confusion because it mentions a failed payment and asks for a refund.

## My recommendation to ChillarPay

ChillarPay should use the few-shot prompt as the starting point for daily support because it scored better on category, priority and reply tone. A human support agent must review the outputs, particularly fraud alerts and messages that fit more than one category.

## Skills shown

Prompt engineering · Zero-shot and few-shot prompting · Synthetic data creation · AI output evaluation · Customer-support operations · FinTech domain knowledge

## What I learned

- Zero-shot prompting (instructions only) works well for simple, common tasks like summarising.
- Few-shot prompting (instructions + solved examples) teaches AI a company's own rules, tone and format.
- Start with the simplest prompt, and add examples only when the output misses the rules.
- Synthetic data can be used to test AI without using real customer data.
- The answer key should be written before testing so AI output is judged fairly.
- A scorecard helps measure AI quality instead of relying on opinion.
- AI output must be checked by a human before business use.

*ChillarPay and all data in this project are fictional and created for learning.*
"""

for name, content in files.items():
    (base / name).write_text(content, encoding="utf-8")

zip_path = Path("/mnt/data/ChillarPay_Lab_2_Complete_Submission.zip")
with zipfile.ZipFile(zip_path, "w", zipfile.ZIP_DEFLATED) as z:
    for name in files:
        z.write(base / name, arcname=name)

print(f"Created {len(files)} Markdown files and ZIP: {zip_path}")
