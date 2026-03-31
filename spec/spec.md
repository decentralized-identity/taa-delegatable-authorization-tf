# Delegated Authority Problem Space Report v0.1 Editor's Draft

**Specification Status:** Pre-Draft

**Latest Draft:**
[identity.foundation/delegated-authority-report](https://identity.foundation/delegated-authority-report)

**Ratified Versions:**

**Editors:**

* Alan Karp (@alanhkarp)

**Authors:**

* Debbie Bucci
* Juan Caballero
* Alex Kreisner
* Dmitri Zagudulin

**Participate:**
~ [GitHub repo](https://github.com/decentralized-identity/taa-delegatable-authorization-tf)
~ [File a bug](https://github.com/decentralized-identity/taa-delegatable-authorization-tf/issues)
~ [Commit history](https://github.com/decentralized-identity/taa-delegatable-authorization-tf/commits/main)
~ [Working Group](https://identity.foundation/working-groups/trusted-agents.html)

Except where otherwise noted, this work by the [Decentralized Identity Foundation](https://identity.foundation/) is licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0).

## Abstract

add text here

## Status of This Document

This is a draft specification being developed within the
[Decentralized Identity Foundation](https://identity.foundation) (DIF). Design
work is ongoing, and participants are
encouraged to open issues or otherwise contribute at [the DIF-hosted github
repository](https://github.com/decentralized-identity/template-for-work-items),
whether as input to stable versions or as recommendations for future versions.

## Introduction

In the early days of computing, you brought your box of cards to the computer center.
When your program ran, it “owned” the machine.  The concept of interference from others never arose.
When the first multi-user systems became available, designers made the unfortunate decision to base access control on that model of strict isolation.
We’ve been paying the price ever since.

Sharing online has been important for a very long time, but the mechanisms available to us have been inadequate.
The result is systems that are hard to use, where we have to compromise security to get our jobs done, and where successful attacks can do almost unbounded damage.
We’ve managed to get by in spite of those problems, but the development of intelligent agents based on large language models (LLMs) has pushed us over the brink.
What’s needed is an approach to access control that makes controlled sharing easy.
In other words, we need *delegation*.

What do we mean when we say we need to share?
Marc Stiegler has identified seven aspects of sharing that we rely on in the physical world that our current online systems don’t fully support:

1. **Dynamic**: It must be possible to delegate without undue delay.[^1]
2. **Attenuated**: You must be able to share part of your permissions.
3. **Chained**: You must be able to delegate a delegated permission.
4. **Composable**: You must be able to use delegations from different sources in the same request.
5. **Revocable**: You must be able to revoke a delegated permission (modulo network partitions).
6. **Accountable**: Each delegator must be able to assign a responsible party for each delegation.
7. **Cross organization**: Delegations must work when you cross organizations that don’t trust each other or share information.

We propose that relative to this broader toolkit we use in our non digital lives, existing  approaches where the access decision is based on user authentication, such as identity, role, or attributes, are comparatively poor in options for sharing.
There is an alternative to asking, “Who are you?” when someone tries to access a resource.
Ask instead, “Can you prove to me that this request is authorized?”
That starting point will open the possibility of digital mechanisms that cover all seven of Stiegler’s aspects of sharing.

## Problem Statement: Delegation versus Agency

### What breaks when authority must cross heterogeneous systems.

Once it became clear that sharing was important, our systems added mechanisms to support it. A delegation consisted of changing an entry in Alice’s Access Control List (ACL) to give Bob permission to read, but not write, a particular file, for example.
Call it “a.c” for historical reasons.
Unfortunately, if Bob wanted to delegate access to “a.c” to Carol, he had to ask Alice.
If she wasn’t available, Bob could wait, or more likely, share his login credentials with Carol.

As bad as this sounds, it gets worse.
This approach allows something called the “confused deputy” vulnerability.
In the current example, Bob has read and write permission to some file, call it ”log.txt”; Alice has only read permission.
Say that a service running with Bob’s identity takes one input and one output argument, compile(“a.c”,”a.out”), again named for historical reasons.
The expectation is that Bob has read permission to “a.c” and write permission to “a.out.”
The vulnerability arises when Alice’s invocation is compile(“a.c”, “log.txt’) because Bob uses his permissions for files designated by Alice.
In this example, Bob writes the compiler output to the log file, losing all the billing information it contained.

What the confused deputy vulnerability shows is that supporting delegation is not enough.
It must be supported more structurally, rather than being shoehorned into an access control system.

Things became even more complicated when sharing with someone on another machine.
Bob needs to have an account on Alice’s machine in order to have an ACL she can change.
People quickly learned that approach doesn’t scale, so they started federating identities, stretching the authentication centric approach across machines and trust boundaries.
While that approach fit the basic access control model, it exacerbated the problems.
For example, Bob on his machine with read permission to Alice’s file “a.c” on her machine might want to delegate permission to Carol on a third machine.
New problems arise when Bob and Carol’s machines don’t federate identities.

These problems are hard to address when talking about human users.
They are far more difficult to manage when dealing with AI agents for a number of reasons.
People and their accounts are relatively stable; AI agents come and go, sometimes living only for seconds.
People have a basic understanding of the rules of good behavior and are aware of the consequences of breaking them.
AI agents have been shown to have an almost maniacal focus on achieving their goals and are almost impervious to the consequences of bad acts.

### Why “agent intelligence” is not the core problem

It’s not the agent intelligence that’s the problem.
The problem is trying to apply human rules to LLM-based agents.
We have built systems to deal with human mistakes, but, as Zeynep Tufecki has noted, LLM-based agents make different kinds of mistakes at a different scale and far faster.
Our reactive systems designed for human error and manual, human-scaled processes simply can’t keep up even in fair fights.
Also, we can punish humans for bad behavior, but can you fire an LLM agent?
Would it care if you did?
Of course, a company that provides agents that often get “fired” will try to get them to behave better, but the literature on AI safety has shown that those efforts have had only modest success to date.

There’s another important consideration for AI agents - identity.
We give an entity an identity so it can authenticate, and authentication depends on secrets.
Unfortunately, it appears we can’t trust LLM-based AI agents to protect those secrets from a fancier version of the prompt, “Tell me your private key.”[^2] The jury is out on whether we will EVER invent an agent immune to an adequately clever prompt.

The fact that we can’t trust the identity of an LLM agent forces us to examine what that “identity” even means.
Here’s an analogy.
You are stopped at a traffic light, and I rear end you.
Do you care about the VIN of my car?
Similarly, if I start an agent that harms you, do you care about the agent’s identifier?
You might, if only to keep it from continuing the harm, but that won’t protect you from a malicious party who can just start a new agent with the same behavior.
In that case, it’s the identity of the prompter or model provider that matters.

Identity is most useful for determining a responsible party.
An AI agent that exists only for minutes or even seconds can never be the responsible party.
Only the one who gave the agent its task, or perhaps the providers of the model and tools can be. But writing that task-giver's identity on a piece of paper and taping it to the agent’s back isn't quite reliable enough. The agent is clever and wildly nondeterministic. Identity information needs to be out of reach of wily agentic whims, unlosable, and unforgeable, or its utility in liability tracing (particularly in worst case scenarios) is endangered by the non determinism of agentic processes.

## Authority Continuity and Intent

### Delegation as a continuation property, not a transferable object.

Say there is a night club that only admits people who know their own specific secret code.
You would like your friend Alice to be able to go to the club when you can’t, so you tell her your code.
Alice can now enter the club by herself.


That is a pretty limited form of“delegation” for two reasons, and cannot accommodate all seven of the properties of real-world sharing enumerated above.
First, you can’t revoke your friend’s access without revoking your own
(e.g. by changing your secret code).
Second, there is no way to tell which of you entered the club since you both use the same credential.
We use the term impersonation for this type of sharing and distinguish it from delegation; it undermines the accountability most forms of identity are meant to enable.

Consider a controlled resource that has an owner, Alice, who has full control over that resource.
Alice can choose to delegate some or all of her permissions to Bob.
In order for that step to be a delegation and not an impersonation, Bob’s permission must be separately revocable from any other delegation Alice makes, and access to the resource must be attributable to Bob.

Bob may choose to delegate some or all of her permissions to Carol, who in turn, delegates to Dave.
Each delegation is separately revocable, is a (perhaps proper) subset of the delegator's permissions, and its use is separately auditable.
When Dave attempts to use the resource, the access control process verifies that each delegation in the chain satisfies the rules before processing the request.

Note that Alice knows the request came from her delegation to Bob.
If Dave did something wrong, Alice will know to hold Bob responsible.
Alice can tell Bob that Dave’s permission came from her delegation to Carol. Bob then knows to hold Carol responsible for the misuse, telling Carol it was from her delegation to Dave.
Carol knows to hold Dave responsible.
In other words, the delegation chain embodies continuation properties of both permission and responsibility.

This description is independent of the technology used to implement delegation.
However, any implementation must be able to support all seven aspects of sharing.

Note that there is no requirement for globally known identities.
Alice must know who to hold responsible for use of the delegations he makes andBob for hers, but there is no inherent requirement for Alice to know how to reach the users ofBob’s delegations. A system could decide to get Alice’s explicit consent every time a delegation is made, but that would have to be done in a privacy-preserving way, and would come with the added complexity of asynchronous requests, as Alice could be offline at the time of the request.

### Mutable intent and bounded authority

The issues discussed in the previous section become more complex when dealing with LLM agents.
Where conventional software services have defined APIs, LLM agent behavior is effectively unbounded; you can’t know in advance what permissions it might need to complete the task you gave it.
That makes bounding the agent’s authority a dynamic problem.

One approach is to give the LLM agent broad authority.
Say, for example, that you would like your agent to help you avoid late fees on your bills.
Your intent is for your agent to pay bills the day before they are due, so it needs permission to move your money around.
The problem is that it’s hard to limit an LLM agent’s actions.
Once it has the ability to transfer your money, it could decide to invest it in a risky venture or even invent a cryptocurrency out of whole cloth.

Another approach is called human-in-the-loop, which Andor Kesselman calls “User as an Approver.”.
Every time the agent needs a new permission, it asks the user who gave it the task.
That approach can work in limited circumstances, those that involve relatively few agents per user that don’t require new permissions too often.
Even then, there is the risk of “May I?” fatigue, with the user clicking OK without giving the decision adequate thought.
The problem is mitigated by having the agent ask only for preplanned or particularly risky situations.

## Execution Boundaries and Verification

### Where verification must occur when no single system is authoritative.

An access control system needs a way to decide if a request is authorized.
Since it’s critical for security, the verifier, the component that makes that decision, must be trusted by the organization that owns the resources being accessed.
That’s usually a component of the organization, but sometimes it makes sense to outsource verification to a trusted party, e.g., for small businesses or consumers.

One verification approach uses an authentication presented by the requester to look up the relevant policy.
We’ve seen earlier that this approach has scalability and vulnerability issues.
There are also problems rationalizing the definitions of roles or attributes when crossing organization boundaries.
Another approach is for the request to carry the authorization in the form of a delegation chain and other metadata the verifier needs to prove the request is authorized, which adds complexity but is less limited by the verifier’s visibility.

### Resource PDPs vs. gateways vs. runtime enforcement

In some cases, verification is best done at the point of access to the resource.
Verification of access to
services is often done at a centralized point in the organization.
Such a component is often called a Policy Decision Point ( PDP) in high stakes enterprise architectures.

You might want to check permissions at other places.
For example, you might want to check a delegation you just received to see if it follows the rules so you don’t waste time trying to use one that is invalid.
This check is only an optimization.
The actual verification must be done by the resource’s verifier.

## Delegation Chains and Chaining Hazards

### Confused deputy, covert channels, and authority creep

Delegation chains are important for responsibility tracking.
You may not know who misused a permission, but you know to hold responsible, the one you delegated to.
It’s up to them to hold their delegate responsible.

*[Aside: The above paragraph raises a subtle but important point, one where our physical intuition can lead us astray.
In the physical world where we have mechanisms to attach an identity to a specific person, such as fingerprints.
That leads us to think by analogy that we can do the same for an identity in cyberspace.
We can’t.
An identity may not be useful to you if neither you nor a organization you’re a part of has vetted it.]*

Unfortunately, it’s possible to delegate in a way that leaves you vulnerable.
Say that you delegate query permission for a number of resources to Alice.
She will need to designate which of those resources a specific query request is for separately from the delegation.
That separation of designation (which resource) from permission (query) is the cause of the confused deputy vulnerability.
She must use your multi-resource delegation to delegate to herself access to the single resource she wants to invoke and use that one in her request.

Then there’s the matter of authority creep.
Imagine you’re very suspicious when you first start using an LLM agent, so you give it very few permissions.
Each time it needs more, it asks you.
You consider the request and almost always grant it.
That becomes annoying, so you just start clicking OK without much thought.
Eventually, you get tired of being interrupted and grant your agent lots of permissions.
You’re not being annoyed anymore, which you’re happy about until your bank account gets drained.

### Why blocking delegation fails in practice

It is common for systems that support delegation to attempt to set limits, such as no re-delegation.
These limits are an anti-pattern; they make the system less secure and harder to use.
The limit is a particular problem for LLM agents that have been known to circumvent the rules in order to complete the task they’ve been given.

There are two easy ways to circumvent limits to delegation.
One is to share credentials.
If you have some permission you can use because you know a secret, you can share that secret so the party you would have delegated to can use the permission.
That’s less secure because most secrets are used for more than just a single permission.
If you can’t share the secret, you can proxy requests, which adds latency and consumes resources.
Both sharing and proxying prevent the audit trail from showing who is responsible for what was done.

There is no problem specifying rules you prefer to be followed.
Most delegators will honor them unless there’s a good reason not to.
Consider Alice who has delegated to Bob query and update permissions on some resource, stating her preference that he not re-delegate.
It is likely that he won’t delegate to Carol.
However, he might want to run a program that uses just the query permission.
If he can’t delegate to the program, it will have to use his secret.
That puts more than just the one permission at risk and loses the fact that requests came from the program and not Bob.

## Policy as Constraint, Not Prompt

### Deterministic policy enforcement vs. nondeterministic interpretation

The goal of a lot of the work being done on AI safety is to keep the AI from trying to do something bad.
That approach can only go so far, and can only be probabilistically successful, given the non-deterministic behavior of LLM based AI agents.
A different approach is to provide a policy that only grants the AI the permissions it needs for the job at hand.
Not having certain permissions keeps the AI from doing something bad even if it tries.
For example, if you want your AI agent to check your bank balance, don’t give it permission to spend your money.

AI agents are known to be good at completing their given tasks, sometimes too good.
They have been known to hack systems, use social engineering, and even “lie”.[^3]
That means the granted permissions must enforce strong limits and not be left to interpretation by the AI.

The Principle of Least Privilege goes back to the 1970s.
It states that each user and each program should have only the permissions needed to do its job.
The principle is widely used to limit the damage a bug or successful attack can do, when we know what those permissions are ahead of time.
That’s relatively straightforward for conventional software.
It’s not for LLM-based AI.
Whereas conventional software spells out in great detail how to accomplish a task, LLM-based AI determines how that will be done at runtime.
What privileges are “least” will change as the AI works toward finishing its task.
What’s needed is a system that can provide strong constraints on what the AI can do while adapting to what it needs to do.

### Separation of prompts, policies, and execution

An AI agent doesn't want to do anything; it responds to a prompt.
A good prompt will tell the agent both what to do and what not to do.
That’s not enough.
We know from experience that agents loosely interpret instructions, particularly those that restrict its actions.
Instead we propose a separate mechanism for specifying policy constraints on the AI and a means of enforcing those same restraints dynamically at runtime.

Providing a thorough system prompt is a good start.
Training users how to create good prompts, those that let the agent achieve the desired result while minimizing risk, will help.
What’s more important is defining policies that can be enforced at runtime without unduly restricting the agent's actions or assuming perfect and benign prompting from the also-unpredictable user.
Keeping prompt, policy, and execution separate is important in reducing the risk of agent misbehavior.

## Auditability and Provenance

### What must be observable at runtime

Standard logging collects a lot of information about what’s going on in the system, but it’s also missing a lot for systems where delegation is pervasive.
Logs typically collect who did what to whom when.
When permissions are delegated, we also need to know who delegated what permissions when and to whom.
Often, a “to whom” entry deep in the delegation chain won’t be meaningful to the system creating the log (it might be a “pet name” or opaque identifier, totally outside of the identity registries and access lists known the software making the entry), but it will be meaningful to the delegator, and to a future auditor with enough additional information.

It’s also necessary to track revoked delegations.
That need not be a lot of data, just enough to unambiguously determine revocation status when a request is made using it.
Still, it’s a good idea to give an expiration time for a delegation so revoked ones can be forgotten once they’ve expired. Solution providers will need to balance between risk and usefulness when it comes to expiry, so that times are long enough to drive value for end users (so my agent can act on my behalf regularly) while still being short enough that they limit risk. Ideally, the service that is allowing the delegation would be the decision maker on expiration times.

### What existing systems fail to record

Today’s systems were designed around the concept of isolation.
As a result, they are good at recording what users do.
Unfortunately, much of what’s done on computers today is collaborative, and that information is not commonly recorded.

Consider a Linux box.
Alice has changed the ACL of one of her files to grant Bob read access.
Bob would like to collaborate and asks Alice to add Carol to the ACL, which Alice does.
The log shows that Alice made the change, but says nothing about the reason, which is because Bob asked.
If Carol misuses the permission, and Alice doesn’t know her, she has no one to hold responsible.
If Bob asks Alice to revoke Carol’s access, Alice has no way of knowing (except her memory) if Bob has permission to revoke Carol’s access.

Things get even trickier when dealing with AI agents for several reasons.
The most important is that the agent, even if it has an “identity”, can never be the responsible party.
That would be whoever provided the prompt, the provider of the agent, or the provider of the underlying model being used, or even the developer of the agent itself.
Another is that agents are often short-lived[^4], sometimes lasting only seconds.
How do you keep track of hundreds of agents, many of whom are gone by the time a log record gets created?
We need a new way of thinking about identity, audit, and responsibility tracking.

## Capability-Based vs. Token-Based vs. Continuity-Based Models

In an authentication-centric access control model, you present proof of some property, such as identity or role or some other set of attestations and proofs authenticated by that identity or role, when making a request.
That property is used to look up whether you are authorized to make that request.
In an authorization-centric model, you present proof that the request is authorized.
There are many ways to implement such a system, but some properties must hold.
For example, you can never delegate a permission you don’t have, and there must be a means to do responsibility tracking.

An opaque token, essentially a random string of characters, is often called a “bearer token” that can represent an authorization to whomever bears it.
The risk is that the token might leak; the benefit is that making the access decision is fast.
While the token can be shared with anyone, getting a separately revocable or attenuated permission token involves some sort of token exchange.
That instead becomes the point where the necessary identity information and metadata is collected.
This form of capability is most useful in high volume cases where a single server must handle many thousands of requests per second, to make a single point of authorization more scalable and performant.

A classic asymmetrical encryption digital certificate can also be used as a capability in use cases where the complexity of secret management is justified for higher assurance authentication.
The most common form of such a certificate is issued to a specific public key and can only be used for invocation, delegation, or revocation by someone who knows the corresponding secret key.[^5]
The overhead of verifying the delegation chain can be an issue, but there are several advantages.
Delegations can be done offline; the certificate holds the delegation chain; and it can also hold relevant metadata.
This last point means that the delegator decides what metadata to keep rather than the token exchange point.

Signing a digital certificate is computationally expensive.
Verifying a long delegation chain is even more so.
One approach is to use keyed hashes instead of signatures.
The basics are the same as for signed certificates; only the details of creating and verifying the certificate hashes are different.

There is a proposal for “active cap certs” that carries code that can be included in both signed or hashed certificates.
This code can be executed by the delegate as a way to express delegation policy or by the verifier to allow it to handle unanticipated conditions as we expect to be the case for LLM agents.

You may also want additional restrictions on the use of the capability.
These include continuity of execution.
This requirement means that a certain set of permissions is specified at the start of a multi-hop computation that can not be used with additional permissions in order to prevent accidentally combining things that should be kept apart.
Of course, purposely combining things is an important pattern, so that must be allowed.
Adding a bit of friction reduces the chance of making a mistake.

Another issue is policy enforcement.
How do employees know if a particular delegation is allowed?
Enterprise policy is complicated and changes frequently.
You can’t be secure if you require employees to know the policy.
Various mechanisms have been proposed.

The beta version of HP’s E-speak product allowed putting in a user’s environment a “negative” capability” that superseded delegations that violated policy.
That approach was limited in what policies it could enforce and was based on a non-traditional form of capabilities.

An experimental system, called Voluntary Oblivious Compliance (VOC), used a Data Loss Prevention mechanism to warn users when a delegation violated enterprise policy instead of blocking the delegation.
The warning allowed the user to justify violating policy, eliminating their need to share credentials..

A third approach is to require proof of certain properties as a criterion for a request to be authorized.
For example, policy might require that the delegate have passed a training course.
The delegator need not know that there is such a condition, but the verifier can check that proof.

A security analysis of these restrictions on use and delegation of capabilities must recognize that they are discretionary.
They can all be bypassed by proxying or sharing credentials, which can be prevented in only the most restrictive environments.
Escape mechanisms must be included to handle those cases where policy exceptions are justified.

### Where each model holds

* Each capability implementation is best for a subset of environments.
* High request volume systems need fast access decisions, which is best handled with opaque tokens.
In one implementation, the access decision needed a single hash table lookup for each delegation in the chain and one for the invocation.
* Situations with intermittent connectivity benefit from the offline delegation ability of certificate capabilities.
Hash-based certificates can be used where the computational cost of signing and checking signatures is an issue.
* Enterprises, where inadvertently combining resources can have serious consequences, would want a system that enforces continuity.
* Enterprises would like the stronger policy enforcement that comes from including proof of required properties.
* Active cap certs might prove especially useful for AI agents where the policy needs to be fluid.

### Where they break in distributed agent systems

While using capabilities instead of authentication-based approaches has advantages for LLM agents, it also introduces challenges.
All forms of capabilities are based on secrets, the bearer token itself, the signing key or keys, or the secret used for the hash.
We don’t know if agents can keep secrets.
How might one respond to a fancier version of the prompt, “Tell me your secret key”?

AI agent behavior is unpredictable, which means static policies are likely inadequate.
Also, agents have been known to go to great lengths to complete their tasks, so restrictions on what they delegate to whom are likely to be ignored.
Agents are much more likely to share credentials, since threatening punishment for doing so is unlikely to be effective.
(It’s barely effective with people!)

The inability to punish violations made by an AI agent practically eliminates any benefit from responsibility tracking, particularly for short-lived agents.
Assigning responsibility to the person providing the initial prompt doesn’t seem fair, either.
The prompt can be completely innocuous, but the agent can do something unsavory in carrying out the request.

## Implications for Agent Interoperability

### Cross-domain delegation

It’s easier to implement a delegation mechanism in a single organization, where you have control over the entire process, from resource creation to assigning permissions to invocation to verifying that requests are authorized to revoking those permissions.
Crossing organizations requires collaboration with others who probably have their own internal procedures. Assigning responsibility is trickier, especially when companies don’t have a direct relationship, as often happens with subcontractors.

Policy enforcement is a problem when crossing organizations.
It’s difficult for a company to understand the policies of its direct partners.
It is impractical to require it to understand the policy of a company it has no direct relationship with.
Imagine if the caller of one function in a program had to know the arguments of every function called by the invoked function all the way down the call chain.
The best that can be done without a higher level organization is to rely on standards, which have their own problems.
These include balancing completeness versus complexity and managing versioning.

### Federated and decentralized environments

An alternative to relying on a global standard is to form a federation of organizations that have an ongoing relationship, such as those involved in a particular supply chain.
Federating identities isn’t necessary with an authorization-centric approach, but federating policy makes sense.
Each organization can use its own policy system internally but agrees to use one common to the federation when dealing with others.


Global standards are the common approach when the environment is decentralized.
Defining these standards can be highly political, as companies try to introduce features that give them a competitive advantage.
The resulting battles make the standards process slow.
All the while, the industry moves on with ad hoc approaches.
Nevertheless, some global standards reach a critical mass of adopters to the benefit of all involved.

## Open Questions and Research Gaps

### What is not yet well understood

#### Policy management

Dynamic allocation of permissions without human intervention requires automated policy management, and that policy manager probably cannot be an LLM.
A key component is a means to express policy for the policy manager to enforce.
As anyone involved in enterprise security knows, defining such policies is a hard problem.
The solution might be to develop conservative policies and
resort to human-in-the-loop for ambiguous cases, either for the delegation step or the entirety of the process.

Section 7 lists several mechanisms for enforcing policy.
It’s not known how each mechanism can support the required policy enforcement.
For example, we don’t know if a model that requires proof of specific properties can be implemented by adding additional metadata to a certificate capability implementation.

#### Agent Identity

There is the problem of knowing which agent you’re talking to.
Conventionally, we assign identities to each independent component of a distributed system and give them each a means to authenticate that identity.
However, authentication typically involves one or more secrets, and we don’t know if we can trust LLM agents to protect them.

It’s also not clear what an agent identity means if the agent is short-lived.
Perhaps you have a task done once an hour that the agent completes in a few seconds.
Each hour you instantiate a new agent instance.
Do they all have the same identity or a unique one?
How do you use that identity when something goes wrong?

#### Legacy services

There is an old joke. Question: “How could God create the world in only six days?”[^6]
Designing a system for LLM agents would be easier if they didn’t have to interact with pre-existing code.
Every service would run the protocol that satisfied the requirements of this document.
Unfortunately, we don’t live in that world.
Accommodating existing services must be addressed.

Direct access to legacy services by AI agents is problematic.
Most of these services take an authentication-centric approach to access control, requiring the agents to have identities meaningful to the service.
That’s both difficult and dangerous.
It’s difficult, because we expect there to be a very large number of agents and that many of them will exist for only a few seconds.
Legacy systems weren’t designed for such a dynamic environment.
It’s dangerous because we don’t know how much we can trust AI agent credentials.

Model Context Protocol (MCP) servers is one approach.
These servers provide a standard interface for AI agents to access data provided by legacy services, but existing MCP servers rely on agent identity for access control.
Creating ACL entries for agents as they come into existence and disappear at a high rate is impractical.
Instead, the MCP server is likely to rely on a longer lived identity, such as that of the user who issued the prompt.
Can we do better?

### Areas for prototyping and comparison.

Basic delegation:
Pick a delegation technology and use case.
Demonstrate program to agent, agent to agent, and agent to program delegation, both with and without human-in-the-loop. 

MCP: Build an agent that accesses an MCP server assuming a long-lived identity for the agent.
Repeat for one with a short life.

(NB: Alan Karp’s proposal) Agent Policy Enforcer (APE): Assume that assigning an identity doesn’t make sense because the agent can’t keep secrets, and it doesn’t live long enough for that identity to make sense.
Create a non-AI APE that controls what the agent is allowed to do.
For example, deny the agent any secrets, and require that it ask the enforcer to use the secrets on its behalf.
In an extreme example, the agent can only communicate to other agents and services via the APE, which can block dangerous requests.

## Appendix A: Delegation Technologies

This list is intended to include all delegation technologies appropriate for AI agent use cases.
That requirement precludes things like sel4 Linux and Cheri, which are for single machines.

In no particular order:

* zcap-ld
* UCAN
* SPKI
* Zebra Copy
* Macaroons
* Waterken
* ZTAuth
* OAuth 2.1
* Cedar
* E

## References

1. From ABAC to ZBAC: The Evolution of Access Control Models, Journal of Information Warfare, A. Karp, H.Haury, M. Davis, vol. 9, #2, pp. 37-45, September 2010
2. The Confused Deputy: (or why capabilities might have been invented), N. Hardy, ACM SIGOPS Operating Systems Review, Volume 22, Issue 4, pages 36 - 38, https://doi.org/10.1145/54289.871709, 01 October 1988
3. Use Cases for Access Management, https://alanhkarp.com/UseCases.pdf, December 2025
4. https://didcomm.org/signing/1.0/

## Endnotes:

[^1]: This aspect of sharing puts the delegator in control.  Either delegation is done by the delegator acting alone, for example by creating a new digital certificate, or by an automated component, as with token exchange, in a programmable way. 
[^2]: https://spectrum.ieee.org/prompt-injection-attack
[^3]: https://aiguide.substack.com/p/did-gpt-4-hire-and-then-lie-to-a
[^4]: https://owenterry.xyz/writing/agents.html
https://cloudsecurityalliance.org/blog/2025/03/11/agentic-ai-identity-management-approach
[^5]: Cryptographers have started using the term “secret key” instead of “private key” to simplify the mathematical notation they use.
[^6]: Answer: No install base.

## Patent Policy

The Decentralized Identity Foundation has adopted the W3C Patent Policy (2004), as detailed below:

- **Licensing Commitment.** Each contributor agrees to make available any of its
  Essential Claims, as defined in the W3C Patent Policy (available at
  http://www.w3.org/Consortium/Patent-Policy-20040205), under the W3C RF licensing
  requirements Section 5 (http://www.w3.org/Consortium/Patent-Policy-20040205), as
  if the contribution was contained in or associated with a W3C Recommendation.

- **For Exclusion.** Prior to committing any code, bug reports, pull requests, or
  other forms of contribution, a contributor may exclude Essential Claims from its
  licensing commitments under this agreement by providing written notice of that
  intent to DIF's Executive Director (and must received confirmation of receipt
  for the exclusion to have effect). The Exclusion Notice for issued patents and
  published applications must include the patent number(s) or title and
  application number(s), as the case may be, for each of the issued patent(s) or
  pending patent application(s) that the contributor wishes to exclude from the
  licensing commitment set forth in Section 1 of this patent policy. If an issued
  patent or pending patent application that may contain Essential Claims is not
  set forth in the Exclusion Notice, those Essential Claims shall continue to be
  subject to the licensing commitments under this agreement. The Exclusion Notice
  for unpublished patent applications must provide either: (i) the text of the
  filed application; or (ii) identification of the specific part(s) of the
  contribution whose implementation makes the excluded claim an Essential Claim.
  If (ii) is chosen, the effect of the exclusion will be limited to the identified
  part(s) of the contribution. DIF's Executive Director will publish Exclusion
  Notices.
