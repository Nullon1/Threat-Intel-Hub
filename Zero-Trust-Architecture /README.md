# Zero Trust Architecture
 
## Part 1 — The Problem Before Zero Trust
 
### 1. Hook — The Day Credentials Stop Being Yours
 
Imagine this... Monday morning a normal employee starts the day, opens their laptop, checks emails, connects to the company VPN and everything looks normal.
 
Username?
Correct
Password?
Correct
MFA?
Approved
VPN?
Connected
 
From the organization's perspective, everything looks fine. The user is authenticated and access is granted but there is 1 small problem...
 
The person behind that login is not the actual user, it's an attacker and I need to tell u that this scenario is not some futuristic Hollywood-style attack. It's actually 1 of the most common ways attackers enter organizations today.
 
They don't always need to exploit some crazy zero-day vulnerability, Sometimes they just need:
 
- leaked password
- stolen session cookie
- phishing page
- compromised endpoint
and suddenly, they have something much more valuable than an exploit. Maybe u ask what? I gonna tell you they have identity and this is what attackers need.
 
4 years security teams built their defenses around 1 main idea:
 
- Keep attackers outside the network.
- Build a strong perimeter.
- Protect the firewall.
- Monitor inbound traffic.
- Create a trusted internal network.
assumption was simple:
 
Outside = Dangerous
Inside = Trusted
 
and 4 a long time, this model made sense and many organizations around the world relied on it but it worked well when everything important was inside the corporate network, employees were working from company devices, applications were hosted internally, and remote access was rare.
 
The network itself was considered the security boundary but then... world changed.
 
Users started working from everywhere, applications moved to the cloud and employees started using personal devices. SaaS (if u don't know what is it pls search about cloud) platforms became part of daily operations.
 
The "internal network" stopped being a single controlled place so the boundary disappeared & attackers noticed that.
 
a stolen credential doesn't care about ur firewall also it doesn't care if the attacker is physically outside the organization.
 
bcz from the system's point of view, attacker is no longer outside they are using a valid identity and this creates a very uncomfortable question:
 
If an attacker already has valid credentials, should we still trust them? This is the exact problem that led to Zero Trust Architecture.
 
This is not about :
 
"How do we build a stronger wall?"
 
but:
 
"What if someone is already inside?"
 
### 2. Traditional Security — The Castle Model
 
Before talking about Zero Trust, we need to understand the security model that came before it.
 
The traditional approach was built around the idea of a secure perimeter.
 
Think of an old castle there are walls, there is a gate and there are guards. Everyone outside the castle is suspicious but everyone inside is considered safe.
 
The security strategy is mostly focused on protecting the entrance.
 
Internet
↓
Firewall
↓
Internal LAN
 
Once someone passed the gate... then they were inside the trusted zone and somehow we can say u fucked up.
 
This model created a concept called:
 
Implicit Trust
 
Meaning: "If you are already inside, we assume you are allowed to be there." and this assumption became a problem bcz attackers don't always break the door sometimes they steal the key. Imagine an attacker gets access through a phishing attack, they compromise an employee account. now they connect through VPN.
 
Traditional security sees:
 
User: John
Password: Correct
Location: VPN
Authentication: Success
 
Decision: ALLOW
 
But it doesn't ask:
 
- is this really John?
- is this device healthy?
- does this behavior make sense?
- why is this user downloading 20GB of sensitive data at 3 AM?
The authentication happened but the actual trust decision was missing which is most important part.
 
Another problem with this model was lateral movement. Once attackers got inside, moving around the environment became much easier bcz nobody cares about internal network and systems trust each other.
 
Something like:
 
Compromised Laptop
↓
File Server
↓
Domain Controller
↓
Critical Systems
 
attacker didn't need to attack every system from outside. They were already inside the castle now they just needed to move. This is where the old security mindset started breaking.
 
The question was no longer:
 
"How do we stop attackers from entering?"
 
bcz sometimes they will enter and u cant do anything about it. the better question became:
 
"How do we limit what happens after they enter?"
 
### 3. Threat Model — Assume Breach
 
Before understanding Zero Trust, we need to understand the assumption behind it bcz Zero Trust is not just a collection of security products, u must look it different, it's a different way of thinking.
 
The foundation is a simple idea:
 
Assume the attacker is already inside
 
Sounds pessimistic? Maybe but security models are built around realistic assumptions not optimism.
 
Zero Trust starts with a different threat model.
 
Instead of assuming:
 
Internal Network = Safe
External Network = Dangerous
 
It assumes:
 
1. request can be malicious
2. identity can be compromised
3. device can be infected
4. network can be hostile
Let's break this down.
 
**Credentials can be stolen**
 
Passwords are no longer proof of identity, attackers can obtain credentials through:Phishing
 
- Credential dumping
- Password reuse
- Data breaches
A correct password only proves one thing: Someone knows the password, it does not prove who is behind the keyboard.
 
**Endpoints can be compromised**
 
A user might have a valid account but the device they are using could already be infected.
 
Example:
 
                     User authentication:
✓ Correct password
✓ MFA approved
                      Device:
✗ Malware detected  (✗ means exist)
✗ Suspicious process running
✗ Security agent disabled
 
Should access still be allowed?
 
Zero Trust says:
Not automatically
 
**Insider threats exist**
 
The threat is not always an unknown attacker from the internet sometimes the identity itself can become the problem.
 
A valid account performing unusual actions should still be investigated bcz:
 
Valid identity ≠ Valid behavior
 
**Networks are hostile**
 
Zero Trust removes the idea that being inside a network automatically creates trust.
 
- corporate LAN
- home network
- public WiFi
- cloud environment
All of them are just locations & location alone should not decide trust.
 
**Trust is temporary**
 
This is probably the most important concept. Traditional security often thinks:
 
Authenticate once
↓
Trust continues
 
Zero Trust thinks:
 
Request happens
↓
Evaluate context
↓
Make decision
↓
Monitor continuously
↓
Re-evaluate
 
Trust is not a permanent status,  It is a decision made at a specific moment and this changes everything.
 
The goal is not:
 
"Create a network where attackers cannot enter."
 
bcz realistically that network does not exist.
 
The goal is:
 
"Even if someone gets access, how much damage can they actually do?"
 
### 4. Identity As The New Perimeter
 
So if the network is no longer the security boundary then what is it?
 
The answer: Identity
 
In traditional security:
 
Network Location
↓
Trust
↓
Access
 
Being inside the network gave you advantage.
 
But in Zero Trust:
 
Identity
+
Device
+
Context
+
Risk
+
Policy
↓
Trust Decision
↓
Access
 
The user identity becomes one of the most important security signals but identity alone is not enough.
 
Because again attacker can steal identity so Zero Trust does not say:
 
"Trust identity."
 
it says:
 
"Identity is one piece of evidence."
 
a request should answer multiple questions:
 
1. Who are you?
2. What device are you using?
3. Is this device secure?
4. Where are you connecting from?
5. What resource are you trying to access?
6. Does this behavior match normal activity?
7. What is the current risk level?
Only after evaluating these signals should access be granted.
 
This is the fundamental shift:
 
                     Old security:
Trust first
Verify later
                      Zero Trust:
Verify first
Trust temporarily
 
and this is where Zero Trust actually begins not with MFA or with VPN replacement or with buying another security product, It starts with changing one important assumption:
 
Being inside the network does not make you trusted
 
**Security Insight**
 
The biggest mistake in modern security is protecting only the entrance. Attackers don't always break in
sometimes they simply walk in using someone else's identity.
 
Zero Trust was created because modern attacks no longer only target systems, they target trust itself.
 
## Part 2 — How Zero Trust Actually Makes Decisions
 
### 5. What Is Zero Trust?
 
Now that we understand the problem, let's talk about the solution but before that there is one common misunderstanding about Zero Trust.
 
A lot of people think:
 
"Zero Trust means nobody gets access."
 
Or:
 
"Zero Trust means blocking everything."
 
That's not what it means. Zero Trust is not about removing trust completely in fact It's about removing implicit trust.
 
In traditional security, trust was often based on one thing: where are you?
 
Internal Network
↓
Trusted
↓
Access granted
 
Internet
↓
Untrusted
↓
Blocked
 
Simple thing but also... tooooo fucking simple.
 
But Zero Trust changes the question.
 
Instead of asking:
 
"Are you inside the network?"
 
It asks:
 
"Based on everything we know right now, should this specific request be allowed?"
 
A request in Zero Trust is not treated as automatically safe because of:
 
- Network location
- VPN connection
- Corporate IP address
- Previous authentication
Every request needs evaluation. Think about it like this: you enter a building.
 
Traditional security: the guard checks your ID at the front door.
After that, you can move anywhere inside.
 
Zero Trust: the guard checks your ID then checks which room you need. Checks if you are allowed there , if your badge is still valid. Checks if your behavior makes sense.
 
And keeps watching.
 
The goal is not to make access impossible. The goal is to make unauthorized access harder and limit the damage if something goes wrong.
 
### 6. Core Principles Of Zero Trust
 
Zero Trust is based on several principles. These principles are what separate it from traditional security models. Let's go through them.
 
**Never Trust, Always Verify**
 
This is probably the sentence most people associate with Zero Trust. But what does it actually mean?
 
It means: no user, no device, no application, no network should automatically receive trust and successful login does not mean unlimited access.
 
The identity does not change from one request to the next but the situation around it can and Zero Trust cares about the situation more than the identity.
 
We'll see exactly how that plays out in a moment, when we look at how the Policy Engine actually decides.
 
**Least Privilege Access**
 
Another important principle: users should get only the access they need nothing more.
 
Traditional environments often end up like this:
employee joins a company, gets access to some resources, changes role, gets more access, changes departments, keeps old permissions.
 
User
↓
Too many permissions
↓
Large attack surface
 
This creates a problem called Privilege Accumulation.
 
Zero Trust tries to reduce this
 
A user should have the minimum access, for the minimum time, for the minimum resource.
 
Example: a developer needs access to a production serve so should they have permanent administrator access? Probably not a better model is :
 
Request Access
↓
Verify Identity
↓
Check Device
↓
Approve Temporary Access
↓
Complete Task
↓
Remove Access
 
Because every unnecessary permission is another opportunity for an attacker.
 
**Assume Breach**
 
We already discussed this in the Threat Model but it is also one of the core principles.
 
The mindset changes from:
 
"How do we prevent every compromise?"
 
to:
 
"How do we reduce the impact of compromise?"
 
Because perfect prevention does not exist and its so important for u to understand this concept if u interested in Blue Team and Cyber Defense.
 
Someone will click the phishing email or reuse a password or idk for example Someone will install something suspicious.
 
The important question is: what happens after that?
 
**Continuous Verification**
 
Authentication is not a one-time checkpoint. It's the start of an ongoing process, not the end of one.
 
A user's risk level can shift minutes after they log in — a device gets infected, a location changes, behavior turns suspicious. We'll walk through exactly how that works in Part 3.
 
### 7. Request Flow — What Happens When Someone Wants Access?
 
Now we understand the ideas but how does Zero Trust actually work? Let's follow a request.
 
Imagine a user wants to access an internal application so user sends a request:
 
User
↓
Application Request
 
The request does not directly reach the application instead, it goes through Zero Trust components.
 
Identity Provider
↓
Policy Engine
↓
Risk Evaluation
↓
Policy Decision
↓
Policy Enforcement Point
↓
Application
 
**Step 1 — Authentication**
 
First, the system needs to know: who is requesting access? this is where Identity Provider comes in
 
- Active Directory
- Azure AD / Entra ID
- Okta
- Other IAM systems
The user proves their identity, usually with:
Password, MFA, Certificate, or a Hardware token.
 
But remember: authentication only answers "Who r u?"
 
It does not answer "Should you access this resource right now?"
 
**Step 2 — Collect Context**
 
After authentication, the system collects additional information. For example:
 
- Identity — Who is the user?
- Device — Is the endpoint healthy?
- Location — Where is the request coming from?
- Time — Does this make sense?
- Risk — Is there suspicious activity?
- Resource — What are they trying to access?
This context becomes the input for the next step. The actual decision

