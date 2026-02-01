# Contact Form – Protection & Consent (Short Guide)

## 1. Protection / Security

- **Cloudflare** is a strong first layer (DDoS, WAF, basic bot filtering).
- Still add some **backend basics**:
  - Server-side validation (length limits, format checks, strip HTML/JS, avoid SQL injection).
  - Simple rate limiting (per IP and/or per email).
  - Optional bot protection: Cloudflare Turnstile / CAPTCHA / honeypot field.
  - Always use **HTTPS** and don’t store form data longer than necessary.

*(These points relate to the general security and data protection principles of the GDPR, e.g. integrity and confidentiality under Art. 5(1)(f) GDPR.)*

---

## 2. Consent / Legal Basis (GDPR-style thinking)

For a **simple contact form** that is only used to reply to messages:

- You typically **don’t need a separate consent checkbox**.
- Legal basis is usually:
  - *Contract / pre-contractual steps* under **Art. 6(1)(b) GDPR**, and/or
  - *Legitimate interest* (communication with users/customers) under **Art. 6(1)(f) GDPR**.
- But you **should** show a short notice like:

  > By submitting this form, your data will be used to process your request  
  > (legal basis: Art. 6(1)(b) and/or Art. 6(1)(f) GDPR).  
  > For details, please see our Privacy Policy.

- In your **Privacy Policy** (information duty, cf. **Art. 13 GDPR**), explain:
  - Which data you collect (e.g. name, email, message, IP).
  - For what purposes and how long you store it.
  - That security providers (e.g. Cloudflare) may process IP addresses/logs.

If you also use the data for **marketing / newsletters / profiling**:

- Then you **do need** a **separate, explicit consent checkbox** based on **Art. 6(1)(a) GDPR**, e.g.:

  > ☐ I agree to receive marketing emails and understand that my data  
  > will be used for this purpose (Art. 6(1)(a) GDPR).

---

## 3. Quick Checklist

- [ ] HTTPS enabled  
- [ ] Cloudflare active (optionally with Turnstile or similar)  
- [ ] Server-side validation + rate limiting  
- [ ] Short processing notice under the form (Art. 6(1)(b)/(f) GDPR)  
- [ ] Privacy Policy linked and up to date (Art. 13 GDPR)  
- [ ] Extra consent checkbox **only** if data is used beyond answering the inquiry  
      (e.g. marketing under Art. 6(1)(a) GDPR)
