# Presenting at ICSSP 2022: Privilege Escalation Attacks on DevOps Pipelines

I'm excited to share that my research on DevOps security will be presented at the International Conference on Software and System Processes (ICSSP) 2022!

---

## Conference Details

**Conference:** International Conference on Software and System Processes and International Conference on Global Software Engineering (ICSSP'22)
**Location:** Pittsburgh, PA, USA
**Dates:** May 20–22, 2022
**Website:** https://conf.researchr.org/home/icssp-2022

**My Presentation:**
- **Session:** Security Track
- **Format:** Research Paper Presentation
- **Co-authors:** Dr. Lotfi ben Othmane (Iowa State University) and Altaz Valani (Security Compass)

---

## About the Conference

ICSSP is a premier international conference focused on software and system processes, bringing together researchers and practitioners to discuss the latest advances in software process improvement, DevOps practices, and software engineering methodologies. The conference provides a platform for sharing empirical research and practical experiences in software development processes.

This conference is particularly important for DevSecOps research as it bridges the gap between academic research and industry practice, focusing on real-world applicability of software process improvements.

---

## Paper/Presentation Title

**Title:** Privilege Escalation Attack Scenarios on the DevOps Pipeline Within a Kubernetes Environment

**Co-authors:**
- Nicholas Pecka, Iowa State University
- Dr. Lotfi ben Othmane, Iowa State University
- Altaz Valani, Security Compass

---

## What I'll Be Presenting

### Abstract

Companies are misled into thinking they solve their security issues by using tooling that is advertised as aligning with DevSecOps principles. This paper aims to answer the question: Could the misuse of the DevOps pipeline subject applications to malicious behavior?

To answer the question, we designed a typical DevOps pipeline utilizing Kubernetes (K8s) as a case study environment and analyzed the applicable threats. Then, we developed four attack scenarios against the case study environment: maliciously abusing the user's privilege of deploying containers within the K8s cluster, abusing the Jenkins instance to modify files during the continuous integration, delivery, and deployment systems (CI/CD) build phase, modifying the K8s DNS layer to expose an internal IP to external traffic, and elevating privileges from an account with create, read, update, and delete (CRUD) privileges to root privileges.

### Key Topics

I'll be covering:

1. **Threat Modeling of DevOps Environments**
   A comprehensive threat model of a DevOps environment using Kubernetes, analyzing potential security weaknesses in the software supply chain

2. **Four Attack Scenarios**
   Practical demonstrations of privilege escalation attacks targeting different components of the DevOps pipeline (Kubernetes DNS, Jenkins CI/CD, networking, and storage)

3. **Mitigation Techniques**
   Proposed protection mechanisms to address the identified threats and secure DevOps pipelines

---

## Why This Work Matters

### The DevSecOps Misconception

Many companies adopt DevSecOps tools believing that using software "advertised as aligning with DevSecOps principles" automatically solves their security issues. This research demonstrates that this assumption is dangerous.

**The Reality:** Simply using DevSecOps tools doesn't guarantee security. Companies need to understand:
- How DevOps pipelines can be exploited
- The importance of proper configuration and access control
- That automation can introduce new attack vectors despite its benefits

### Who Benefits from This Research

**For Security Teams:**
- Understand specific attack scenarios targeting their DevOps infrastructure
- Learn practical mitigation strategies
- Identify security gaps in current DevOps implementations

**For DevOps Engineers:**
- Recognize security implications of pipeline configurations
- Implement least-privilege principles in Kubernetes environments
- Design more secure CI/CD workflows

**For Organizations:**
- Make informed decisions about DevOps security investments
- Understand that tool adoption alone isn't sufficient
- Develop comprehensive DevSecOps strategies beyond tooling

---

## Research Highlights

### Attack Scenario Overview

We demonstrated four distinct privilege escalation attacks:

**Attack 1: Strimzi Data Siphoning**
Leveraging Kubernetes DNS to deploy a malicious application that can access sensitive data from Apache Kafka streams within the cluster.

**Attack 2: Jenkins Build Manipulation**
Compromising the CI/CD pipeline by modifying files during the build phase, potentially affecting all systems using the modified application.

**Attack 3: Kubernetes Networking Exposure**
Exploiting Kubernetes networking objects to expose internal cluster IPs to external users, bypassing intended network isolation.

**Attack 4: hostPath Namespace Breakout**
Escalating from CRUD privileges to root access by exploiting Kubernetes hostPath volumes to escape namespace isolation.

---

## The Journey to This Point

This research emerged from observing a critical gap in industry understanding of DevSecOps security. During my work with organizations implementing Kubernetes and CI/CD pipelines, I noticed a recurring pattern: teams assumed that using modern DevOps tools automatically meant their deployments were secure.

### Key Challenges Overcome

**Building a Representative Environment**
Creating a realistic DevOps pipeline using industry-standard tools (GitHub, Jenkins, DockerHub, Kubernetes) that accurately reflected production environments.

**Threat Modeling Complexity**
Analyzing the intricate interactions between different pipeline components to identify exploitable security weaknesses.

**Balancing Research and Responsibility**
Demonstrating real attack scenarios while providing actionable mitigation strategies to help organizations defend against them.

---

## What to Expect from the Presentation

The presentation will include:

- **Live Environment Demonstration:** Visual walkthrough of the DevOps pipeline case study
- **Attack Scenario Videos:** Pre-recorded demonstrations of each attack scenario in action
- **Threat Model Visualization:** Visual breakdown of the analyzed threat surface
- **Practical Mitigation Strategies:** Actionable recommendations for securing DevOps pipelines

All attack demonstration videos and resources will be made publicly available following the conference.

---

## Conference Preparation

### Presentation Development

This presentation synthesizes months of research into a 20-minute talk that makes complex attack scenarios accessible to both researchers and practitioners. The challenge has been balancing technical depth with clarity—ensuring security professionals can implement the mitigation strategies while researchers understand the methodology.

### Research Resources

All research materials, including:
- Attack scenario demonstration videos
- Complete DevOps environment configuration
- Threat model documentation
- Mitigation implementation guides

Will be shared publicly at: https://github.com/npecka/privilege-escalation-devops-k8s

---

## Looking Forward

I'm hoping to gain from attending the conference:

- **Research Community Feedback:** Validation of our methodology and identification of additional attack vectors
- **Industry Practitioner Insights:** Understanding real-world DevOps security challenges organizations face
- **Collaboration Opportunities:** Connecting with researchers working on software supply chain security
- **Future Research Directions:** Identifying gaps and opportunities for expanding this work

---

## Key Takeaways

**Main Message:** DevSecOps tool adoption ≠ DevSecOps security

Organizations must:
1. Understand their DevOps pipeline attack surface
2. Implement least-privilege principles throughout the pipeline
3. Regularly audit and test pipeline security configurations
4. Not rely solely on out-of-the-box tool security

**Research Contributions:**
- Threat model of Kubernetes-based DevOps environment
- Four validated attack scenarios demonstrating privilege escalation
- Practical mitigation techniques for each identified threat

---

## Can't Attend? Here's How to Follow Along

- **Slides:** Will be posted to my website after the presentation
- **Paper:** Available through ACM Digital Library (DOI: 10.1145/3529320.3529325)
- **Attack Videos:** Publicly available on GitHub
- **Full Research Materials:** https://github.com/npecka/privilege-escalation-devops-k8s

---

## Questions?

Have questions about the research? Want to discuss DevOps security challenges you're facing? Feel free to reach out or leave a comment below!

---

**Following my research?** Subscribe to get notifications when I publish new research, papers, or conference announcements.

---

*This post will be updated after the conference with presentation materials and reflections.*
