+++
date = '2025-05-18'
draft = false
title = 'detection engineering: what matters and can you measure success?'
+++


# a beginning
I have been thinking lately about detection engineering. Specifically, I have been thinking about these two questions as they relate to a detection engineering program:

1. What matters?
2. Can you measure success?

I am not sure I have a good answer to either of these questions yet and there isn't a lot of authoritative information out there. As far as I can tell, detection engineering is a somewhat specialised and relatively new field within cyber security. It seems like most organisations are doing their own thing and only sufficiently mature organisations appear to have dedicated detection engineering teams.

For context, when I talk about detection engineering, I am referring to the apparent evolution that has taken place from the primitive concept of security monitoring. The fundamentals are the same, but the former builds on the latter by incorporating engineering methods and principles such as lifecycle modelling and optimisation as well as DevOps practices like something-as-code and automated testing.

Despite the limited amount of information, some of the concepts I have come across in my research might be constituent parts of good answers and are helpful in their current form. I aim to explore some of those in this post and future posts.

There are several reasons I have been thinking about detection engineering. Firstly, I work in detection and response and it constitute a large part of what I do day-to-day. Secondly, and more importantly, I have come to appreciate the significant way in which it underpins and enables defensive operational awareness and the role it plays in causing the adversary a great deal of pain.

My plan is to start with discussing these questions at a high level and work my way down to the details. But I make no promises to adhere to that plan. To that end, I have one outcome I am ultimately trying to achieve on this journey:
> Fix a bit of the murky thought cloud in my head by distilling it into writing that makes a little bit of sense.

If a by-product of this is developing guidance that is simple, non-prescriptive, and easy to apply in practice for thinking about, maturing, and managing a detection engineering program, then I will have exceeded my own expectations.

As I progress, it is likely that I will discover new information and change how I see things. I will revisit and update or make new posts when that happens. I will also probably be making a lot of implicit assumptions about what you know, this is selfishly written for me.

# what matters?
I first want to start with purpose. Without purpose, it is difficult to be coherent and make sense of why and how we do things. Ignoring the potential philosophical implications of that statement, I take the liberty of assuming it is true. Moving forward, I will use the following definition of purpose for detection engineering.

Purpose
: To continuously convert threat knowledge into signals that reveal adversary behaviour in your environment and enable the timely disruption of their progress toward malicious goals.

This definition is broader than most I have come across. I like it for two main reasons:
1. It highlights natural and important connections with three other key functions in the wider defensive program; **threat intelligence**, **threat hunting**, and **incident response**.
2. It alludes to *signal detection theory*, a well established field of study with methods and results that we can use to better understand the problem space and optimise.

![Connection](/images/connection.svg)

To expand on the first reason briefly:
- Understanding your environment's threat profile is crucial for focusing your attention on surfacing signals that are most relevant. You have a limited amount of resources and they should be used accordingly. If your most salient threat is a ransomware gang, you likely need to focus on a slightly different set of signals compared to a nation state. They go about their business in different ways and this will be evident in the data.
- The process of converting knowledge into signals in this context sounds suspiciously like a threat hunt. In my view, back testing a detection rule during development is threat hunting. It may not be particularly sophisticated as far as threat hunting goes, but if your rule is threat informed then you are nonetheless doing a threat hunt.
- Disrupting the adversary before they can achieve malicious goals is the ideal scenario for incident response. If you detect the adversary after they have achieved their goals, your detection has failed and your organisation and responders are probably not having a good time.

On the second reason, a detection rule is nothing more than a type of *binary classifier*. For simplicity, and without much loss of generality, if we restrict our definition of detection rule to those that are search based, a detection rule $r$ can be viewed conceptually as a composition of two functions:
- A signal extraction function $s$ that maps data $x\in X$ to signals.
- A decision function $d$ that maps those signals to a binary threat classification.

That is to say, the rule $r$ computes a threat label $y\in (\text{threat}, \text{no threat})$ as:
$$
y = r(x) = d(s(x))
$$

This abstraction is helpful for highlighting how separate features of a rule can be evaluated and optimised independently. We can improve fidelity through $s$ and tune the decision boundary through $d$. For example, $s$ may catch encoded PowerShell commands and $d$ includes logic to determine which are malicious.

There are existing tools and methods that we can borrow from signal detection theory for measuring the performance of binary classifiers, understanding the nature of tuning them, and recognising their limitations. Taking advantage of these tools and methods can also provide insights that will allow us to avoid mistakes that lead to common issues in the detection and response space. For brevity, I will refrain from listing them. If you have spent time triaging alerts then you know.

Given this purpose, I think that there are ultimately three variables that matter; coverage, quality, and capacity. Anything that you do as part of a detection engineering program should contribute in some way to improving one or more of them. I suggest further that the objective of a detection engineering strategy should be to maximise a function $f$ of these variables subject to some constraints, for example budget or head count. It is a constrained optimisation problem.

<div>
$$
\begin{aligned}
\text{max:} \quad & f(\text{coverage}, \text{quality}, \text{capacity}) \\
\text{subject to:} \quad & \text{constraints}
\end{aligned}
$$
</div>

![Strategy](/images/strategy.svg)

## 1. Coverage
Coverage refers to the extent to which your detection rules collectively target surfacing the range of signals that are known to be created by threat actors. A *coverage gap* presents the adversary with an opportunity to achieve their goals without detection. There are many ways to think about what this means. The contemporary example is facilitated by MITRE ATT&CK through tactics and techniques. You can build a map between your rules that surface tactics and techniques and the ATT&CK library to visualise or measure your coverage.

Given limited resources, you can constrain the library by prioritising tactics and techniques based on your threat profile. [This post](https://detect.fyi/detection-engineering-lifecycle-an-integrated-approach-to-threat-detection-and-response-54de5bf17dba) outlines a prioritisation method along these lines. Of course, tactics and techniques are not the only signal we aim to surface, but they do sit at the top of the [Pyramid of Pain](https://www.attackiq.com/glossary/pyramid-of-pain/). 

![Coverage](/images/coverage.svg)

Since your rules are dependent on data from your environment, coverage also entails in some way that you have the data that is required for your rule set. **You cannot detect what you cannot see** and seeing is having the data available for your rules to function.

It is also important to acknowledge the fact that the adversary is not stationary. They adapt. The threat landscape is constantly evolving. Your rule set should be equally adaptable. If your rule set lags behind changes to your threat profile then you end up with a coverage gap. Continual assessment of coverage is necessary.

## 2. Quality
Quality reflects mainly how well your rule is documented, promotes triage efficiency through context, and it's performance. However, I think quality is a complex variable with more contributing factors. To expand on just the three highlighted:
1. **Documentation**: we want the rule to be accompanied by documentation that at least facilitates easily understanding:
    * The intention, goal, or objective of the rule - what signal is it trying to surface?
    * Why the signal matters - is this worth the time that your incident response team will spend investigating?
    * How the rule works - what data is required and how is signal extracted?
    * Expected triage or response actions - how do you arrive at a verdict and how should you respond?
    * Steps to test or trigger the rule - how can you verify that it works?
 
    Palantir have a blog post outlining their [Alerting and Detection Strategy Framework](https://blog.palantir.com/alerting-and-detection-strategy-framework-52dc33722df2) that covers this ground in more detail.
2. **Context**: we want the rule to provide sufficient information about the triggering signal in an alert so that triage time can be minimised by:
    * Avoiding excessive manual investigative effort required to reach a verdict.
    * Automating the required response where possible.
3. **Performance**: we want the rule to surface signals that constitute real threats and not noise. There are several performance metrics associated with binary classifiers we can lean on here. I have to stress that not all metrics are equal and it is crucial to understand their strengths and weaknesses. This is especially true when signals are rare and the base rate fallacy is at play. The most accurate rule can have low precision and a high false positive rate.

## 3. Capacity
At the end of the day, your triage team can only do so much. All else being equal, there will be a certain alert rate that your team is capable of handling before they start to experience burnout or make investigative compromises and mistakes. This rate is what I call *capacity*. The alert rate must be carefully matched to capacity.

Jai Minton has a [blog post](https://www.jaiminton.com/internal-blog/high-impact-security-analysis#) that, among other things, walks through the triage process and explains in detail the analyst mindset. In particular, he relates the concept of the iron triangle to alert triage. He says that security analysis can be good, fast, or cheap, but you can only ever pick at most 2 in any given instance. Given the sobering reality of a budget and applying this concept to a triage team collectively, I suggest further that there is actually an upper bound on speed and it can easily be overwhelmed by:
- A poor quality rule set.
- A fragmented detection surface with tools that are insufficiently scrutinised or improperly configured.

When faced with this situation you can either retain your good security analysis, amass alert debt, and blow out all triage and response time metrics, or forgo good security analysis.

![Fatigue](/images/fatigue.svg)

Like quality, capacity is somewhat complex and there are many contributing factors. These include:
- The number of analysts you have. 
- The experience, expertise, and motivation of your analysts.
- The maturity of your triage processes.
- The analysis tools available.
- How much of your alert triage and response is automated.

How many of these factors the detection engineering program shares responsibility for will depend on the broader cyber security operating model. But it should be acutely aware of all of them.

# can you measure success?
Many parts of theory, science, and professions are concerned with measuring things. There are good reasons for this. Measurement allows us to test assumptions, understand the effects of change, make informed decisions, and build bridges that don't collapse. If you can measure something, it is easier to manage and optimise. It is also often hard to do, especially when uncertainty plays a large role. Risk, for example, is defined by uncertainty, and entire professions are dedicated to understanding and quantifying it.

There are good reasons to measure the success of a detection engineering program. But what does success even mean in this context? I think this is actually quite a hard question to answer. If we base success on the established purpose, then success implies at least identifying the presence of all adversaries before they can achieve their objectives.

The problem is that you don't know the ground truth. You can't know a priori if an adversary is present or not and you can't easily prove their absence. If your detection rules don't surface adversary behaviour it doesn't necessarily mean one isn't there. Absence of evidence is not necessarily evidence of absence. You could just have bad detection rules. And that's fine, because think of all the room for improvement.

On the other hand, it can be very easy to tell if you have failed (assuming sufficiently competent incident response):
- If your data ends up for sale on the dark web, you have probably failed.
- If your business operations are disrupted by ransomware, you have probably failed.
- If you end up in the news for a breach, you have probably failed.
- If your incident response team discovers attacker activity that should have triggered a detection rule but didn't, you have failed.

What these examples serve to highlight is an asymmetry. While failure tends to be visible and costly, success is quieter, harder to grasp, and may go unrecognised. This asymmetry and the intangible nature of success poses a challenge. I suspect that the best we can do in practice to measure success is to use a tailored combination of:
1. Experience based heuristics.
2. Continuous testing.
3. Proxy metrics.

## 1. Experience Based Heuristics
Under this category I place all of the maturity models and adjacent frameworks. These seem to be largely constructed from aggregated experience. They cannot tell you directly if you are succeeding. But they can provide guidance that allows you to situate and orient your program so that it is moving in the right direction (at least according to the collective experience of seasoned practitioners or experts).

Some notable references in this space include:
- Elastic's [Detection Engineering Behavioural Maturity Model](https://www.elastic.co/security-labs/elastic-releases-debmm). I am partial to Elastic because they understand that trying to do detection engineering without a normalised schema is like trying to find your asset inventory in the Library of Babel with a magnet and a pair of dowsing rods.
- Ryan Stillion's [Detection Maturity Level model](https://ryanstillions.blogspot.com/2014/04/the-dml-model_21.html). A great read that pre-dates the release of ATT&CK for Enterprise and covers a lot of ground on, and adjacent to, tactics and techniques.
- Kyle Bailey's [Detection Engineering Maturity Matrix](https://detectionengineering.io/). It is comprehensive at a high level but concise enough to fit on a single page.

## 2. Continuous Testing
To know if something is working you should test it. If something changes, you should test it again. If some time has passed since you last tested it, you should also test it again. In this case there are two primary methods that I think apply:
1. Unit testing.
2. Adversary emulation.

### Unit Testing
For each rule in your rule set you should know how to generate signals that the rule will surface. You should create these signals on a regular basis in a clean way in your environment to trigger the rule and confirm that the alert triggered correctly. Ideally this process is automated and bypasses triage.

### Adversary Emulation
One team, two team, red team, blue team, or purple team. Whatever you want to call it, pretend to be the adversary and do the thing. The more the better. If you can't do it, get someone else to do it. Did you detect all the things? Great! Otherwise, you have work to do.

There is a recent trend among ransomware groups where they will provide their victims with a detailed report outlining how they gained access after payment. This sounds cool, almost like a legitimate business service. But you can get the same outcome with more certainty, less pain, and for fewer gold coins from people who are not financially motivated criminals. Plus, there are all the other kinds of adversaries that have no incentive to tell you how they did the thing.

## 3. Proxy Metrics
These are statistics you can calculate, sometimes easily, but that are not directly measuring detection engineering success. Rather, you might be able to infer something about success from them. Examples include:
- Rule set coverage indicator.
- Mean time to detect.
- Mean time to triage.
- Alert fatigue indicator.

None of these are going to be particularly informative and they are almost always confounded by factors that you might not be aware of or that are outside your remit. Be careful when calculating and interpreting these statistics.

# to summarise
I think there are three variables that matter:
- Coverage
- Quality
- Capacity

A detection engineering program should aim to maximise a function of these variables subject to organisational constraints.

Defining and measuring success is hard because success is somewhat intangible and there is an asymmetry between success and failure. It is likely the best we can do for now is to use a combination of experience based heuristics, continuous testing, and proxy metrics.