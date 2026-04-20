# The Identity Concentration Problem

*Your identity provider is not one system among many. It is the substrate the business runs on. The security question stopped being about how strong your login is, and became about what an attacker gets when the login eventually fails.*

---

At 9:17 on a Tuesday morning, a bookkeeper at a twenty-eight-person company clicks a link in what looks like a DocuSign notification from a client she works with every week. The page asks for her Microsoft 365 password. She types it. Her phone buzzes with the multi-factor authentication push she has tapped through a thousand times before, and she approves it because she was expecting a DocuSign. Nothing visible happens on the screen. She closes the tab and goes back to reconciling expenses.

At 9:18, somewhere else, an attacker is signed in as her.

By 9:45 they have read three months of her inbox, noticed she has Helpdesk Administrator rights she did not remember being granted, and used those rights to reset the multi-factor authentication on the company owner's account. By 10:30 they have the full contents of SharePoint, which includes the client list, the vendor contracts, and four years of payroll. By 11:15 they have set a rule on the owner's mailbox that forwards any message containing the word "wire" or "invoice" to an outside address and deletes it from his Sent folder, so he never sees the replies. By 12:02 they are inside the company's QuickBooks Online, because the owner signs in through his Microsoft account, and QuickBooks trusts the Microsoft identity without asking any further questions.

The bookkeeper goes to lunch at 12:30. The company will figure out something is wrong on Thursday, when a client calls to confirm a wire request that no one at the company remembers sending.

This story is not about a weak password. The password was strong. It is not about missing multi-factor authentication. That was on. It is about what a single account in a modern productivity suite is actually connected to, and what an attacker gets when they get in. The security incident is not the stolen credential. The security incident is everything downstream of that credential, which in this case turned out to be the entire operational surface of the business.

Most small companies have not caught up to this shift. They still think about identity the way they thought about it a decade ago, when email was a system, file storage was a different system, the accounting software was a third system, and a compromised login to any one of them was bad but bounded. That world is over. For the typical small or mid-sized business in 2026, Microsoft 365 or Google Workspace is not one system among many. It is the substrate every other system sits on top of. The question is no longer whether your authentication is strong. The question is what an attacker gets when authentication fails, because eventually, somewhere, for someone, it will fail.

This piece is about that question, and what to do about it.

## How we got here

Fifteen years ago, the company directory was one system of several. The file server lived in a closet and had its own logins. The accounting software ran on one PC in the back office and had its own password, usually written on a sticky note under the keyboard. The CRM was either on paper or on a separate web service with a separate account. Email ran on an Exchange server that talked to Active Directory, and if you knew what Active Directory was, you were probably the person who set it up.

A compromised Active Directory account in that world was a bad day. It was not the end of the business. An attacker who stole a password got email and file shares. They did not get the accounting system, because the accounting system lived in a different silo. They did not get the CRM, because the CRM was somewhere else with its own credentials. Most of the business was held together by the fact that every important system had its own front door, and getting through one did not open the others.

Then two things happened at roughly the same time, and neither of them was framed as a security shift at the time.

The first was that productivity moved to the cloud. Email, file storage, chat, video meetings, and document collaboration consolidated into a pair of suites. Almost every small and mid-sized business in the English-speaking world now runs on Microsoft 365 or Google Workspace. The directory that authenticates the user to email is the same directory that authenticates them to the file store, the chat platform, the meeting platform, the shared calendar, and the collaborative documents. That was already a concentration shift, but it was not the big one.

The big one was single sign-on. Every other business application the company uses, the CRM, the accounting system, the payroll provider, the password manager, the help-desk tool, the e-signature platform, the expense software, now offers the option to let the user sign in with their Microsoft or Google account instead of maintaining a separate password. This is sold as a convenience feature, and it is. It is also an architectural decision. What it says is: the identity in the productivity suite is now the identity in every other system. If you can prove you are that identity, you are authenticated everywhere.

The shift was not announced. It happened one integration at a time, over the course of several years, as a series of small decisions about convenience. Every time someone clicked "Sign in with Microsoft" on a new SaaS tool, the company's identity surface quietly extended one more system further out. Nobody sat in a meeting and said "let us bet the entire business on the integrity of one login." That is the effect, but that was never the conversation.

So here is the picture in 2026. A small business that runs on Microsoft 365 or Google Workspace has a single identity layer that, in practice, controls access to:

- All email for everyone in the company, historical and ongoing.
- All files in the shared and personal cloud storage, which for most companies now means essentially every document the company has ever produced.
- All chat history and meeting recordings.
- Every SaaS application the company uses that offers single sign-on, which is most of them, including the accounting system, the CRM, the password manager, the HR platform, and the expense software.
- The ability to reset passwords and multi-factor authentication for every other user in the company, if the account has administrative rights.

The word "identity" is still the word we use. What it refers to has completely changed. Most of the security advice that circulates in small-business circles was written for a world where the word still meant what it used to mean. The advice has not caught up. Before the rest of this piece lands, it matters that you see the shift clearly. The single account is not a login anymore. It is the operational spine of the business.

## Why multi-factor authentication is not the answer you think it is

The standard answer to the risk I have just described is multi-factor authentication. You have probably heard this answer from your IT person, from your cyber insurance underwriter, from a compliance questionnaire, or from a conference you attended in 2022. Enable MFA everywhere and you are fine. You can even keep your poorly constructed password. The account cannot be compromised without the second factor. Problem solved.

I want to push back on this gently, because MFA matters and should absolutely be everywhere. The problem is not that MFA is bad. The problem is that "we have MFA" is a finish line in the old model and something closer to a starting line in the current one. The attackers have moved on and left you behind. The defenses sold as "MFA" in 2019 are not defending against the attacks running in 2026. You are not wrong to have enabled MFA. You are, possibly, in the position of thinking you are further along than you are, because the thing you turned on no longer works as well as when it was first recommended to you.

## Here is what has changed.

The most common category of attack against small-business identity in 2026 is not password guessing. It is what the industry calls adversary-in-the-middle phishing, which is a technical term for a conceptually simple trick. The attacker sends a phishing email with a link. The link goes to a site that looks like the real login page for Microsoft or Google, because it is the real login page. Not a copy. The attacker's server sits between the victim and the actual Microsoft or Google login, relaying every keystroke and every response in real time. The victim types their password into what is functionally the real login form, because every byte is being faithfully forwarded. Microsoft or Google sends a multi-factor prompt to the victim's phone, because as far as they can tell, the person is legitimately signing in. The victim taps approve, because they did just try to sign in. Microsoft or Google sends back a session token, which is the little piece of data the browser uses to stay logged in. The attacker's server grabs the session token and keeps it. The victim sees a slightly broken redirect, or a "try again" message, and moves on with their day.

The attacker now has a fully-authenticated session. Not the password and the second factor. The session itself. The session is good for hours or days depending on configuration, and multi-factor authentication was already satisfied to produce it, so the attacker does not need to satisfy it again. They are the user, as far as every system is concerned.

This is not a theoretical attack. Kits that do this end-to-end have names you can look up. Evilginx is the open-source research toolkit that a generation of criminal kits were built on top of. Tycoon 2FA, EvilProxy, Sneaky2FA, and Rockstar2FA are other commodity versions, sold as a service to criminals who do not want to set up infrastructure themselves. A subscription runs a few hundred dollars a month and the technical barrier is roughly the same as running a Shopify store. What used to require a skilled attacker now requires a credit card.

## There are other ways that MFA fails at small-business scale, and they are worth naming.

There is MFA fatigue, where the attacker has a password from a prior breach and fires off push approval requests to the user's phone repeatedly until the user taps approve, either by accident or to make the notifications stop. There is help-desk social engineering, where the attacker calls the managed service provider or the internal IT person, impersonates an employee, and talks them into resetting the account's MFA. There is SIM swap, where the attacker convinces the mobile carrier to move the victim's phone number to a SIM they control, and suddenly every text-message MFA code goes to the attacker instead of the victim. There is token theft from the endpoint itself, where malware on the user's laptop reaches into the browser's storage and extracts the same session token an adversary-in-the-middle attack would steal, without needing the phishing lure at all.

The pattern across all of these is the same. None of them require the attacker to defeat the second factor. They route around it. The industry term for this is MFA bypass, which describes the outcome accurately but obscures the point. The point is that the second factor is a check on authentication, and authentication is not the only thing in the session.

If this is the first you are hearing about adversary-in-the-middle phishing, I want to be clear that you are not behind. The kits became commodity in late 2023. They have been the dominant attack path against small-business Microsoft 365 tenants since roughly the middle of 2024. The reason you have not heard about them in the way you heard about ransomware in 2018 is that they do not produce loud incidents. They produce quiet ones. The attacker gets in, reads the inbox, waits for a wire to be in motion, redirects it, and gets out. The victim finds out later, through the bank, or the client, or the auditor. Most of these never make the news.

So the reframe is this. MFA is necessary, but no longer not sufficient by itself. "We have MFA" is the answer to a question that was being asked in 2018, and the question has changed. The question now is whether the rest of your architecture assumes that authentication is going to hold, or assumes that it eventually will not. If your architecture assumes authentication will hold, you are exposed to the category of attack that is actually running right now against companies of your size.

## The blast radius frame

I want to give you a concept, because once you have it, I believe the rest of the piece is easier to follow.

Blast radius is what an attacker gets when a given account is compromised. It is a shape, not a score. For every account in your organization, there is a blast radius, and it is the answer to the question: if this account is taken over today, what does the attacker now have access to, what can they change, and what can they use the account to reach?

Blast radius is the variable that matters when prevention fails. Most of small-business security spending goes into making prevention work better. Better email filtering, better endpoint protection, more training for the users. That spending is fine. It raises the cost of attacks. It does not reduce the consequence when one succeeds. The reason security architects in larger organizations spend so much time thinking about segmentation, least privilege, and privileged access is that they know a well-run program will still lose accounts occasionally, and the job is to make sure that when it happens, the radius is small enough that the business survives the event. That discipline has not made it into most small-business security thinking, which still treats every successful attack as a failure of prevention to be fixed layering on more.

The table below is a rough map of blast radius by account type in a typical Microsoft 365 or Google Workspace environment. The specific terms differ between the two platforms, but the shape of the problem is the same.

| Account type | What the attacker reads | What the attacker can change | What they can reach next |
|---|---|---|---|
| Standard user (no admin rights) | This user's email, files, and chat history. Access to shared files the user has permissions to. Content in SaaS apps the user has SSO access to. | Only this user's own settings and content. Can set mail-forwarding rules. Can share files outward. Cannot change anyone else's account. | Any SaaS app the user has single sign-on into. Often includes one to three systems with financial or customer data depending on the user's role. |
| Finance or operations user | Everything a standard user has, plus financial systems, payroll, and vendor data. | Can initiate wires if their role allows. Can modify vendor records. Can change payment routing. | The banking platform, the accounting system, the payroll provider. This is the target role for business email compromise. |
| Helpdesk or user administrator | All the user's own content, plus metadata about every other user. | Can reset passwords and multi-factor authentication on other users. Often cannot read other users' data directly, but can assign themselves a role that can, then undo it. | Effectively every other account in the tenant by way of password reset, one at a time. |
| Application administrator | Own content. Full visibility into which applications are connected to the tenant. | Can grant consent to new applications on behalf of the organization. Can change application permissions. Can create new service accounts with elevated permissions. | Any external service the attacker wants to wire in with broad permissions. This is how long-term persistence is established. |
| Global administrator (Microsoft) or Super admin (Google) | Everything. All mail, all files, all chats, all settings, the full audit log, every user's data. | Everything. Can promote other accounts, demote accounts, change tenant-wide security settings, delete users, wipe devices, and disable logging. | Every connected SaaS system, either directly through SSO or by promoting another account to access it. |
| Service account or automation identity | Whatever the service account was granted. Often more than any human account, because nobody wanted to deal with permission errors during setup. | Whatever the service was granted. Often includes writing to mailboxes, modifying files, and running as other users. | Everything the service integrates with. These are frequently forgotten, rarely monitored, and often have no multi-factor authentication at all because they are not human accounts. |

A few things should jump off the table at you.

The blast radius of a global administrator account is effectively the entire business. Not a subset. The entire business. This is the account that should be treated like the master key to the building, and often is not, because it is usually also the account that reads the owner's email.

The blast radius of a helpdesk administrator is almost as bad, by a different path. They cannot read your files directly, but they can reset the password and the multi-factor authentication on any account they have authority over, including the global administrator. From there, every door opens. This is the account type that most small businesses have quietly accumulated, because at some point someone needed to help reset passwords during onboarding, and the role got handed out and never taken back.

The blast radius of a service account is often larger than any human account in the tenant, and is often monitored less carefully than any human account in the tenant. That is an unfortunate combination that attackers understand very well.

The standard user's radius is small in what they control directly, but the SSO column is doing a lot of work in that row. A standard user in a company with heavy SSO integration still provides a path into several systems that were not on the attacker's original list.

Security architecture, reduced to one sentence, is the discipline of arranging your accounts and systems so that the attacker you cannot prevent ends up in a small box rather than a large one. Most small-business tenants are arranged so that the boxes are very large.

## What actually reduces the radius

What follows is not a checklist. It is a set of architectural moves in rough order of how much blast radius they eliminate per unit of effort. I want you to read them as design choices, not as a to-do list, because the point is that they work together. Doing one of them in isolation gives you a third of the benefit. Doing four of them together gives you most of the benefit. I will be honest about which are easy and which are hard at SMB scale.

### Separate administrative accounts from daily-driver accounts

This is the single most valuable change, and it is free. The principle is that no one should be using the same account to read their email and to administer the tenant. The global administrator account should not have a mailbox. It should not be signed into a browser that is also used for browsing the web. It should be used exclusively when administrative action is required, and the rest of the time, it should be signed out.

In practice this means every person who currently administers the tenant gets two accounts. Their regular one, which has no administrative rights, handles email, calendar, files, meetings, and everything else they do during the day. A second account, named something like `admin-firstname@company.com`, has the administrative rights and nothing else. No mailbox, or an empty one that is monitored only for system alerts. No licenses beyond what the administrative role requires. This account exists to be used for ten minutes at a time, occasionally, and to be signed out otherwise.

The reason this matters is that the phishing lure in the opening scenario could not have worked this way. The administrator was reading email in the account with administrative rights. When the session was stolen, the session had administrative rights attached. If the administrator had been reading email in a separate account, the stolen session would have had the blast radius of a standard user, not the blast radius of a helpdesk or global administrator. The attacker would still have gotten in, and they still would have gotten the administrator's mailbox, but they would not have gotten the keys to the tenant. This simple architectural change converts the worst-case event into a merely-bad event.

The cost of this is one extra un-licensed account per administrator, plus the friction of signing into a second account when administrative work is needed. The friction is real but it is a feature, not a bug. It means administrative actions happen deliberately, because a deliberate action is required to enter the administrative context.

### Put break-glass accounts on hardware keys and lock them away

A break-glass account is a dedicated administrative account that exists for one purpose, which is to get you back into your tenant if everything else breaks. It is not used day to day. It is not tied to any individual. Its credentials are stored offline. Its multi-factor authentication is a hardware security key, specifically a FIDO2 key such as a YubiKey or a Feitian ePass, which is a small physical device that plugs into a USB port or taps to a phone. Every modern productivity suite supports FIDO2 keys for administrator accounts, and if yours does not, that is a red flag worth raising with whoever sold it to you.

The break-glass account is excluded from most of the conditional access restrictions that apply to everyone else, so that it can log in from anywhere in an emergency. Its password is long, random, printed on paper, and stored in a sealed envelope in a physical safe. Its hardware key is also in the safe, or in a second safe if the owner is cautious, which they should be. The account is not in anyone's password manager, because password managers can be compromised through the same identity chain this piece is about.

You need at least two break-glass accounts, configured identically, for the same reason aircraft have two engines. The day you need the break-glass account is the day something has gone very wrong, and you do not want to discover that the single one you had was tied to a phone number that no longer works.

This is perhaps the single most impactful control you can put in place, because it gives you one thing nothing else gives you, which is the ability to regain control of your tenant when you have lost it. Every incident response I have been involved in where the client did not have break-glass accounts in place ended up taking twice as long as it should have, because the first several hours were spent trying to find a working administrative path back in. Every incident I have been involved in where the client did have break-glass accounts was, at minimum, recoverable on the timeline the client could afford.

### Put hardware keys on every administrative account, not just the break-glass ones

The next move, in priority order, is to upgrade the multi-factor authentication on every account with administrative rights, including the per-administrator separate accounts from the first section, from the kind of MFA they have now to the kind that does not fall to adversary-in-the-middle phishing.

Most small-business tenants use authenticator-app MFA. The user gets a six-digit code, or a push notification, from an app on their phone. This is the kind of MFA that adversary-in-the-middle phishing defeats cleanly, because the attacker's relay server captures the resulting session regardless of how the second factor was proven. Authenticator apps are better than text messages, but they are not phishing-resistant.

FIDO2 hardware keys are phishing-resistant by design. The cryptography built into them binds the authentication to the specific site requesting it, and an adversary-in-the-middle relay cannot produce a valid authentication on a site other than the one it is pretending to be. This is not a clever configuration. It is a property of the protocol. A FIDO2 key will simply refuse to authenticate to a phishing site, even one that is perfectly relaying to the real site, because the relay site is not the real site and the key knows it.

The cost here is real. A pair of YubiKey 5 keys runs around a hundred dollars, and every administrator needs at least two, because the one they lose on a Tuesday is the one they need on a Wednesday. For a small business with four administrators, that is roughly eight hundred dollars one-time. The configuration work is modest. The benefit is that the phishing attacks actually running against you in 2026 do not work against accounts protected this way. This is the single best price-to-protection ratio available in small-business security right now, and it is underused because the items on the procurement list are less abstract than the attack they prevent.

For very high-value non-administrative accounts, typically the owner, the CFO, and anyone with wire authority, this is worth doing too. The rest of the user base can stay on authenticator-app MFA for the time being, because the cost of hardware keys for fifty users is meaningfully different from the cost for four. The priority is that every administrative account and every financially privileged account is on phishing-resistant MFA. That is the minimum target.

### Restrict where administrative access can come from

Most identity providers will let you set conditions on when administrative accounts can sign in. Conditional access, as the feature is generally called, lets you write rules of the form "administrators can only sign in from a company-managed device" or "administrators can only sign in from this list of trusted network locations" or both.

The version of this that is worth the effort says, roughly: administrative accounts can only sign in from a managed device that has recently passed a health check, from within a defined network, using phishing-resistant MFA. The effect is that even if an attacker manages to steal an administrator's session token, they cannot use it from their infrastructure, because their infrastructure is not a managed device on your network. The token is still valid in theory. In practice, the first time the attacker tries to use it, the sign-in is blocked.

There are tradeoffs here. The administrators cannot do emergency work from a personal phone in a hotel room without advance planning. That is a feature for the ninety-five percent of cases and a friction for the five percent. The break-glass accounts, which are exempt from these rules, exist to cover the five percent. The tradeoff is worth making, because the alternative is that the session token works from anywhere, which is the condition the attackers are optimizing against.

### Revisit who has what administrative role, and take most of them back

In every small-business tenant I have ever been invited into for an assessment, there are administrative roles assigned to accounts that do not need them. The former IT person who left in 2022 still has Global Administrator. The bookkeeper has Helpdesk Administrator because four years ago she needed to help with password resets during an onboarding surge. The marketing contractor has Application Administrator because the intern who set up the analytics tool did not know which role to grant and picked the one that definitely worked. The owner has Global Administrator because everyone has always said the owner should have it, although in practice the owner has never used it and would not know what to do with it if they did.

This accumulation is not malicious. It is the natural drift of a system where adding a permission is always easier than the alternative, and removing one requires someone to notice it and care. The noticing does not happen until something bad forces it to, at which point the rights that should not have existed turn into the path the attacker took.

The fix is a quarterly review, short, not ceremonial. Pull the list of every account that has any administrative role. For each one, ask the single question: does this person currently need this role to do their job? If the answer is no, remove it. If the answer is maybe, remove it and wait to see if anyone complains. If the answer is yes, note when you last confirmed that. The review takes an hour. It should have been happening for the last three years in most tenants, and has not. Start now.

### Monitor the five high-signal events that matter

Monitoring is where most small-business security advice falls apart, because the generic advice is to "monitor your logs," and a small business does not have a security operations center to monitor anything. Fine. I am not going to tell you to monitor your logs. I am going to tell you there are five specific events in a Microsoft 365 or Google Workspace tenant that almost never happen legitimately and almost always happen during an administrative compromise. If you get an alert on only these five, you will catch the vast majority of incidents in time to act.

The five events are: an administrative role being assigned to any account, a new application or service principal being granted broad consent in the tenant, a new inbox-forwarding rule being created on any mailbox, a new federation or trust relationship being added to the tenant, and a sign-in from a country or region where nobody in the company has ever signed in before.

These events are available in the audit log of every major productivity suite. They can be piped to a monitoring tool if you have one, or to an email address if you do not, or to a Slack channel, or to a phone notification. The specific delivery mechanism matters less than the fact that a human looks at the alert within a reasonable time. If your managed service provider is handling security monitoring for you, these are the events to confirm they are monitoring, by name. If they cannot tell you whether they are monitoring them, they are not.

### What this adds up to

Taken together, these moves do not prevent the phishing email from arriving. They do not prevent the user from clicking the link. They do not prevent the session from being stolen. They prevent the stolen session from being catastrophic, and they make the resulting incident visible quickly enough to contain. That is the point. The design target is not perfect prevention, because perfect prevention is not achievable at any price. The design target is limited consequence, and limited consequence is achievable at a price most small businesses can afford.

## The continuity dimension

I have been writing as if the threat model is entirely adversarial, and I want to widen the lens for a minute, because the same concentration problem that makes compromise catastrophic also makes outage catastrophic, and outage is more common than compromise.

Your identity provider will have bad days. Microsoft and Google both have occasional multi-hour outages in various services, and when the service that is down is identity itself, everything that depends on identity is down at the same time. This is not a hypothetical. It has happened several times in the last three years, for each of them, for durations ranging from an hour to most of a business day.

When your identity provider is down, your email is down. Your files are inaccessible. Your chat platform is dark. Every SaaS tool that federates sign-in through the identity provider is locked, even though the tool itself is running fine. Your password manager, if it happens to also federate through the same identity, is locked, so the passwords you would fall back to are behind a door you cannot open. Your help desk system, your accounting software, your CRM. The identity provider is not down because it is attacked. It is down because a bad deployment got pushed, or a region had a networking event, or a certificate expired. It will come back. You are still waiting.

There is a minimum floor of continuity that every small business should have in place, and it has almost nothing to do with backups in the traditional sense. It is about whether the business can operate for four hours with the identity layer unavailable.

The floor looks like this. There is a secondary communication channel that does not depend on the primary identity provider. If you run on Microsoft 365, that might be a Google Workspace account used only for continuity, or a secondary email address on a domain with a different provider, or at minimum a group SMS thread that reaches everyone. Critical documents, meaning the ones you would need to keep the business running through a day of outage, have an offline copy somewhere a person can actually retrieve. The list of things that are critical is short. It is not every file. It is the current client contracts, the current project files in active use, and the information required to reach people and pay bills.

The written playbook for what to do when the identity provider is unavailable does not live in the identity provider. This is the part that trips up most small businesses, because their runbooks live in the same SharePoint or Google Drive that is unavailable. The playbook lives on paper, or on a USB drive in a known location, or in a document on the owner's phone that works offline. It is not long. It says who is the decision-maker during an outage, what the secondary communication channel is, which systems need to be checked first when the identity provider comes back, and who owns each of those checks.

The break-glass accounts from the earlier section have a role here too. If the identity provider comes back but some of your accounts are locked out for unrelated reasons, the break-glass account is the one that can restore order. If you do not have a break-glass account, the first several hours of recovery are spent trying to regain administrative access, which is not where you want to be spending those hours.

This part of the piece is shorter than the section on compromise, and that is deliberate, because continuity for a small business does not require a large investment. It requires acknowledging that the identity layer is a single point of failure for operations and making minimal arrangements for the hours when it is unavailable. Most small businesses have no such arrangement, not because they have considered it and decided not to, but because the question never came up. The question is coming up in this sentence. Consider it, make some arrangements, and move on.

## Where to start

If the piece above is more work than you can absorb in one sitting, which is a reasonable reaction, here is the triaged version. The first list is work that can be done this week by a competent person spending a few hours across several days. The second list is work for the current quarter, most of which requires ordering something, scheduling a change window, or getting buy-in from the rest of the business.

### This week

- Pull the list of every account in your tenant with any administrative role attached. Global admin, helpdesk admin, user admin, application admin, exchange admin, anything with the word admin in it. Write down who each account belongs to, whether the person still works at the company, and whether they have used the administrative rights in the last ninety days.
- For every administrative account on that list, confirm that multi-factor authentication is enabled and note which method. Flag any that are using text-message MFA for upgrade during the quarter. Flag any that have no MFA at all for same-day fix.
- Confirm that at least one break-glass administrator account exists. If none does, create one today. Give it a long random password, store the credentials on paper in a physical safe, and keep it out of every password manager the business uses. Getting to the full version of this control, which is two break-glass accounts protected by hardware keys, is a quarter-scale project. Getting to one break-glass account whose password is not sitting in the same identity layer it is meant to recover is a one-hour project.
- Set up alerts on the five high-signal events described earlier: administrative role assignments, new application consents in the tenant, new inbox-forwarding rules, new federation or trust relationships, and first-time sign-ins from unusual countries. Every major productivity suite has a way to route these to an email address or a chat channel without buying a separate tool. The specific destination matters less than the fact that a human sees them within a few hours.
- Search every user mailbox in the tenant for existing forwarding rules that send mail to external addresses. This finds incidents that have already happened and that nobody has noticed yet. If you find rules that cannot be explained, treat the finding as an active incident and work backward from there.

### This quarter

- Stand up separate administrative accounts for every person who currently does administrative work from a daily-driver account. The new accounts have no mailbox, no extra licenses, and no rights outside the administrative scope. Transition the administrative work onto the new accounts over a couple of weeks. Remove the administrative rights from the original daily-driver accounts once the transition is complete. Budget for one extra license per administrator and a small amount of ongoing friction in exchange for a large reduction in blast radius.
- Order FIDO2 hardware keys for every administrative account and every high-value non-administrative account. High-value means the owner, the CFO or finance lead, and anyone with wire-approval authority. Two keys per account, because the one the user loses on a Tuesday is the one they need on a Wednesday. Enroll them, then require phishing-resistant MFA on those accounts and disable the weaker MFA methods for them once the keys are in place.
- Write conditional access rules that restrict where administrative sessions can originate. The target rule: administrative accounts can only sign in from a managed device that has recently passed a health check, using phishing-resistant MFA. Pilot the rule against a single administrator for a week before enforcing it broadly. The failure mode of a misconfigured conditional access rule is that you lock yourself out of your own tenant, which is why the break-glass account exists and why the pilot step is not optional.
- Run the full quarterly administrative role review, then schedule the next one. For every administrative role assignment in the tenant, ask whether the person currently needs it to do their job. Remove the ones nobody can justify. Document the date of the review so the next one has a baseline, and put the next review on the calendar now rather than hoping someone remembers in three months.
- Write the continuity playbook. It identifies the decision-maker during an identity-provider outage, the secondary communication channel the business will use when the primary one is down, the short list of documents that need an offline copy, and the steps to take when the identity provider comes back online. Store it somewhere that does not require the identity provider to open. Paper, a phone note that syncs offline, or a USB drive in a known location all work. What does not work is a SharePoint document describing what to do when SharePoint is unavailable.

This is not the complete program. It is the subset that moves the blast radius of the worst-case event by roughly an order of magnitude, which is where the effort-to-benefit ratio is most favorable. The remaining items from the piece are refinements once the foundation is in place.

## Where this leaves us

The shape of the security conversation for small businesses needs to change, and it is slowly starting to. The old conversation was about keeping the attacker out. The tools of that conversation were antivirus, firewall, spam filter, and password. The new conversation is about what happens when the attacker is already in, because the modern attack path increasingly ends with a valid session in your identity provider, and the existing tools do not see that session as an attack. The session looks, to every system downstream, exactly like you.

Most small-business security spending is still going into the old conversation. Better endpoint protection, better email filtering, more training. These are fine investments. They reduce the number of times the attacker succeeds. They do not reduce the consequence of the successes that will happen anyway, and they do not address the fact that the successes, when they come, will come through the identity layer and will have the blast radius of the identity layer unless you have done the work to make the radius smaller.

The work to make the radius smaller is not exotic. It is separation of administrative accounts, phishing-resistant multi-factor authentication on the administrative and high-value accounts, conditional access rules that restrict where administrative sessions can originate, a disciplined quarterly review of who has which administrative role, monitoring of the five events that matter, and break-glass accounts stored offline against the day something goes badly wrong. None of these items is new. None of them is expensive. The reason they are not in place in most small-business tenants is that nobody said out loud that they need to be, and the defaults ship without them.

If you take nothing else away, take this. The account that runs your business is worth more than the account that runs your business thinks it is worth. It is not a login. It is the operational spine of everything you do. Treat it accordingly. The work is not large. The consequence of not doing the work is not small.

There is a second half to this problem that I have touched on in the continuity section and want to address more fully in a follow-up piece. If identity is the front door, the question of what survives when the front door is breached, or when the front door is simply unavailable for the afternoon, is the other half of the problem. Resilience is a distinct conversation from security, with its own set of architectural moves and tradeoffs, and it deserves its own treatment rather than a subsection at the end of this one. I am working on that piece next.

For now, start with the week list above. Most of it will take less time than you expect, and by the end of it you will know where your tenant stands well enough to prioritize the quarter list sensibly. The Tuesday morning in the opening of this piece was not a hypothetical. Every week it is somebody's Tuesday morning. The work is about making sure that when it is yours, the bookkeeper's bad day stays the bookkeeper's bad day, instead of becoming the company's.
