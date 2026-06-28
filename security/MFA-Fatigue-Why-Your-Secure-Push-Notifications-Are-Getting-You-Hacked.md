> 📖 **Original article:** [MFA Fatigue: Why Your 'Secure' Push Notifications Are Getting You Hacked](https://www.valtersit.com/guides/security/MFA-Fatigue-Why-Your-Secure-Push-Notifications-Are-Getting-You-Hacked/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

Companies will spend $50,000 a year on a premium Identity Provider (IdP) tier, roll out an authenticator app to the entire org, and proudly declare themselves "unhackable" at the next board meeting. 

Then Kevin in Sales gets his password scraped by an infostealer. At 2:14 AM on a Tuesday, his phone buzzes. He ignores it. At 2:15 AM, it buzzes 30 more times in rapid succession. Bleary-eyed, irritated, and just wanting the screen to go dark so he can go back to sleep, Kevin hits "Approve". 

Congratulations. Your multi-million dollar security perimeter was just breached because Kevin was tired. 

Basic push notifications are not a security boundary; they are an annoyance threshold. If your security architecture relies on a binary "Yes/No" from an exhausted human, it is structurally flawed.

### The Mechanics: Prompt Bombing

This attack vector is known as "MFA Fatigue" or "Prompt Bombing," and it is the primary way threat actors like Lapsus$ and Scattered Spider have been breaching Fortune 500s. 

The attacker already has the valid username and password—usually bought for $5 on a darknet market or phished via a fake Office 365 login page. The only thing standing between them and your corporate VPN is the MFA prompt. So, they script the login portal to fire authentication requests repeatedly. 

Sometimes they don't even rely on pure fatigue. They will hit the user with three prompts, then immediately call the user's phone, spoofing the caller ID to look like the internal Helpdesk. "Hi, this is IT. We're doing an overnight server migration and need you to acknowledge the prompt on your phone to keep your account active."

The user taps "Approve", the attacker gets the session token, and they are in. It relies entirely on human psychology, requiring zero technical sophistication once the password is known.

### The Fix: Kill the "Approve" Button

The modern Zero-Trust approach dictates that you must immediately disable simple "Approve/Deny" push notifications. 

You must enforce **Number Matching**. With Number Matching, the login screen displays a randomly generated 2-digit number. The user cannot simply tap "Approve"; they *must* open their authenticator app and manually type that specific number. You cannot type the number if you aren't the one looking at the login screen. It completely neutralizes MFA fatigue.

For highly privileged accounts (Domain Admins, Global Admins), you should deprecate phone-based MFA entirely. Mandate FIDO2 hardware keys (like YubiKeys). FIDO2 is cryptographically bound to the TLS session and the specific domain being accessed, making it mathematically phishing-resistant.

### The Code & Config

If you are using Microsoft Entra ID (formerly Azure AD), Number Matching is now the default, but if you have legacy policies overriding it, you are vulnerable. Here is the Microsoft Graph API payload to strictly enforce Number Matching, along with application context and geographic location display.

```json
// THE REAL ENGINEER'S WAY (Enforce Number Matching via MS Graph API)
// PATCH [https://graph.microsoft.com/v1.0/policies/authenticationMethodsPolicy/authenticationMethodConfigurations/microsoftAuthenticator](https://graph.microsoft.com/v1.0/policies/authenticationMethodsPolicy/authenticationMethodConfigurations/microsoftAuthenticator)
```

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/security/MFA-Fatigue-Why-Your-Secure-Push-Notifications-Are-Getting-You-Hacked/](https://www.valtersit.com/guides/security/MFA-Fatigue-Why-Your-Secure-Push-Notifications-Are-Getting-You-Hacked/)**
