# Privilege Escalation Attack Scenarios on the DevOps Pipeline Within a Kubernetes Environment

Published research demonstrating how DevOps pipelines can be exploited through privilege escalation attacks, challenging the assumption that DevSecOps tool adoption alone ensures security.

---

## Paper Information

**Title:** Privilege Escalation Attack Scenarios on the DevOps Pipeline Within a Kubernetes Environment

**Authors:** Nicholas Pecka (Iowa State University), Lotfi ben Othmane (Iowa State University), Altaz Valani (Security Compass)

**Published in:** Proceedings of the International Conference on Software and System Processes and International Conference on Global Software Engineering (ICSSP'22)

**Publication Date:** May 20–22, 2022

**DOI/Link:** https://doi.org/10.1145/3529320.3529325

**Status:** Published

**Conference Location:** Pittsburgh, PA, USA

---

## Abstract

Companies are misled into thinking they solve their security issues by using tooling that is advertised as aligning with DevSecOps principles. This paper aims to answer the question: **Could the misuse of the DevOps pipeline subject applications to malicious behavior?**

To answer this question, we designed a typical DevOps pipeline utilizing Kubernetes (K8s) as a case study environment and analyzed the applicable threats. We then developed four attack scenarios against the case study environment:

1. Maliciously abusing the user's privilege of deploying containers within the K8s cluster
2. Abusing the Jenkins instance to modify files during the CI/CD build phase
3. Modifying the K8s DNS layer to expose an internal IP to external traffic
4. Elevating privileges from an account with CRUD privileges to root privileges

**The attacks answer the research question positively:** companies should design and use a secure DevOps pipeline and not expect that utilizing software "advertised as aligning" with DevSecOps principles alone is sufficient to deliver secure software.

---

## Research Context

### The Problem

Organizations are rapidly adopting DevSecOps practices, integrating security into their DevOps workflows. However, many companies fall into a dangerous trap: they believe that simply using tools marketed as "DevSecOps-aligned" automatically solves their security problems.

**The Critical Gap:** There's a significant difference between:
- Using DevSecOps tools (Jenkins, Kubernetes, container registries)
- Having a *secure* DevSecOps pipeline

**Real-World Impact:**
- Development and production environments are increasingly integrated
- Automation provides tremendous benefits but introduces new attack vectors
- A compromised DevOps pipeline can affect every application it deploys
- Software supply chain attacks are on the rise

### Why It Matters

**For the Security Community:**
This research provides concrete evidence that DevOps automation, while beneficial, can create an insecure software supply chain system (SSCS) if not properly configured and secured.

**For Organizations:**
Understanding these attack scenarios helps organizations:
- Assess their actual DevOps security posture
- Move beyond checkbox security
- Implement meaningful security controls
- Protect their entire software supply chain

**For the Industry:**
As DevSecOps adoption accelerates, awareness of these vulnerabilities is critical to preventing large-scale supply chain compromises.

---

## Key Contributions

This research makes the following contributions to the field:

1. **Comprehensive Threat Model of DevOps Environment**
   Development of a detailed threat model for a Kubernetes-based DevOps pipeline using the Strimzi application as a case study, identifying potential attack vectors throughout the software supply chain

2. **Four Validated Attack Scenarios**
   Design and demonstration of four distinct privilege escalation attacks targeting different components of the DevOps pipeline, each with practical implementation details

3. **Actionable Mitigation Techniques**
   Proposed protection mechanisms for each identified threat, providing organizations with concrete steps to secure their DevOps pipelines

---

## Methodology Overview

### Approach

We employed a practical security research methodology combining threat modeling and penetration testing:

**Phase 1: Environment Design**
Created a representative DevOps pipeline mirroring industry-standard practices using commonly deployed tools.

**Phase 2: Threat Modeling**
Systematically analyzed the DevOps pipeline to identify potential security vulnerabilities and attack vectors, focusing on privilege escalation opportunities.

**Phase 3: Attack Development**
Designed and implemented four attack scenarios based on identified threats, validating each attack in the case study environment.

**Phase 4: Mitigation Development**
Developed practical protection mechanisms for each demonstrated attack scenario.

### Tools and Technologies

**DevOps Pipeline Components:**
- **Code Repository:** GitHub
- **CI/CD System:** Jenkins
- **Container Registry:** DockerHub
- **Orchestration:** Kubernetes (K8s)
- **Case Study Application:** Strimzi (Apache Kafka on Kubernetes)
- **Custom Attack Application:** Python Flask application

**Infrastructure:**
- **Hypervisor:** VMware ESXi
- **Hardware:** i7-6700K CPU @ 4.00 GHz, 4 cores, 32GB RAM
- **Virtual Machines:** 4 VMs hosting the complete DevOps environment

**Testing Framework:**
- Kubernetes cluster with APIserver, scheduler, controller-manager, etcd
- Multiple worker nodes with kubelet agents
- Custom Python application for demonstration attacks

---

## Key Findings

### Finding 1: Kubernetes DNS Can Be Exploited for Data Exfiltration

**The Attack:**
An attacker with privileges to deploy containers can create a malicious application that leverages Kubernetes' internal DNS to access sensitive data from other applications within the cluster.

**Demonstration:**
By deploying a custom Python Flask application to the same K8s cluster as the target Strimzi application, the attacker can:
- Access the internal cluster IP of Strimzi
- Connect to Kafka topics
- Siphon sensitive data that should be isolated within the cluster

**Implication:**
Simply deploying applications to Kubernetes doesn't provide adequate security isolation. Internal network segmentation and least-privilege access controls are essential, even within the cluster.

### Finding 2: CI/CD Pipelines Are High-Value Targets

**The Attack:**
An attacker with access to Jenkins can modify build steps to inject malicious payloads into applications during the build phase, before they're deployed.

**Demonstration:**
By authenticating to Jenkins and editing build steps, an attacker can:
- Insert backdoors into applications
- Modify application behavior
- Affect all deployments using the compromised build pipeline
- Potentially impact downstream organizations using the software

**Implication:**
The CI/CD pipeline is a critical control point in the software supply chain. Compromising it allows attackers to inject malicious code into every application the pipeline builds, making it a highly valuable target for supply chain attacks.

### Finding 3: Kubernetes Networking Misconfigurations Enable External Exposure

**The Attack:**
An attacker with ability to modify Kubernetes networking objects can expose internal cluster IPs to external users by adding NodePort or Ingress objects.

**Demonstration:**
By creating a NodePort service object, an attacker can:
- Expose applications intended to be internal-only
- Bypass intended network isolation
- Allow external connections to internal services
- Create unauthorized access paths

**Implication:**
Kubernetes networking is powerful but complex. Misconfigured or maliciously modified network objects can completely bypass intended security boundaries, exposing internal applications to external threats.

### Finding 4: Namespace Isolation Can Be Broken Through Volume Exploits

**The Attack:**
An attacker with CRUD privileges in any namespace can exploit Kubernetes hostPath volumes to escape namespace isolation and gain root access to the host node.

**Demonstration:**
By deploying a pod with a hostPath volume mount, an attacker can:
- Execute into the malicious pod
- Use chroot to access the host node's root filesystem
- Locate Kubernetes configuration files on the host
- Gain cluster admin privileges
- Control the entire cluster, including all namespaces

**Implication:**
Kubernetes namespaces provide logical isolation but not strong security boundaries. HostPath volumes, if not properly restricted, can be exploited to completely break namespace isolation and escalate to full cluster compromise.

---

## Practical Applications

### For Security Practitioners

**Immediate Actions:**
- Audit Kubernetes RBAC configurations to enforce least privilege
- Review and restrict hostPath volume usage across clusters
- Implement network policies to isolate namespaces
- Secure CI/CD pipeline access with strong authentication and authorization
- Monitor for suspicious pod deployments and network configuration changes

**Long-term Strategy:**
- Implement comprehensive DevOps pipeline threat modeling
- Establish security gates throughout the CI/CD process
- Regular penetration testing of DevOps infrastructure
- Security training for DevOps engineers on Kubernetes security

### For DevOps Engineers

**Configuration Best Practices:**
- Use Kubernetes Pod Security Standards (PSS) to restrict hostPath volumes
- Implement network policies for namespace isolation
- Configure Jenkins with role-based access control
- Use admission controllers to enforce security policies
- Regularly audit service account permissions

**Tooling Recommendations:**
- Deploy policy enforcement tools (OPA/Gatekeeper)
- Implement runtime security monitoring (Falco)
- Use static analysis for Kubernetes manifests
- Integrate security scanning in CI/CD pipelines

### For Organizations

**Strategic Considerations:**
- DevSecOps tool adoption must be accompanied by secure configuration
- Security of the DevOps pipeline itself is as important as application security
- Insider threats and compromised credentials are significant risks
- Software supply chain security requires defense in depth

**Investment Priorities:**
- Security training for DevOps teams
- Pipeline security auditing and testing
- Access control and identity management
- Security tooling for container and Kubernetes environments

---

## Attack Scenario Details

### Attack 1: Retrieve Information from Kafka Topics via Kubernetes DNS

**Prerequisites:** Ability to deploy containers in the same namespace as target application

**Attack Steps:**
1. Deploy malicious Flask application to Kubernetes cluster
2. Locate target Strimzi application's internal cluster IP
3. Access malicious application's web UI from external network
4. Configure malicious app to connect to Strimzi using internal cluster IP
5. Retrieve and exfiltrate sensitive data from Kafka topics

**Impact:** Potential compromise of sensitive data within Kafka streams

### Attack 2: Manipulate CI/CD Build Process via Jenkins

**Prerequisites:** Access to privileged Jenkins account

**Attack Steps:**
1. Authenticate to Jenkins instance
2. Navigate to target build job
3. Edit build steps to insert malicious payload
4. Trigger build job execution
5. Modified application with backdoor gets deployed to Kubernetes
6. Access application through installed backdoor

**Impact:** Installation of persistent backdoor affecting all future deployments

### Attack 3: Expose Kubernetes Internal Services Externally

**Prerequisites:** Ability to create networking objects in target namespace

**Attack Steps:**
1. Authenticate to Kubernetes cluster
2. Locate target application's internal cluster IP service
3. Create NodePort service object pointing to internal application
4. NodePort assigned in range 30000-32767
5. Internal application now accessible from external network
6. Connect to application using external IP:NodePort

**Impact:** Unauthorized external access to internal applications

### Attack 4: Kubernetes Namespace Breakout via hostPath

**Prerequisites:** CRUD privileges in any namespace

**Attack Steps:**
1. Deploy malicious pod with hostPath volume mounted to host root (/)
2. Execute into malicious pod
3. Use chroot to access host node's root filesystem
4. Locate Kubernetes configuration files (kubeconfig)
5. Use host's kubectl with admin credentials
6. Gain full cluster admin access
7. Compromise applications in other namespaces

**Impact:** Root access to host node and cluster admin privileges

---

## Demonstration Resources

All attack scenarios were recorded and are publicly available:

**GitHub Repository:** https://github.com/npecka/privilege-escalation-devops-k8s

**Included Resources:**
- Complete DevOps environment configuration files
- Attack scenario demonstration videos
- Step-by-step attack reproduction guides
- Mitigation implementation examples
- Threat model documentation

---

## Limitations and Future Work

### Limitations

**Environment Scope:**
This research focused on a specific technology stack (GitHub, Jenkins, DockerHub, Kubernetes). While these are widely used, other CI/CD tools and orchestration platforms may have different vulnerabilities.

**Attack Scenarios:**
We demonstrated four specific attack scenarios, but the threat surface of DevOps pipelines is much broader. Additional attack vectors likely exist.

**Mitigation Testing:**
While we proposed mitigation techniques, extensive real-world testing of these mitigations across diverse environments was beyond the scope of this research.

### Future Research Directions

**Expanded Attack Surface:**
- Research additional attack vectors in cloud-native CI/CD pipelines
- Investigate attacks targeting serverless deployment pipelines
- Explore GitOps-specific security vulnerabilities

**Automated Detection:**
- Develop automated detection methods for DevOps pipeline attacks
- Create behavioral analysis tools for CI/CD anomaly detection
- Build security metrics for DevOps pipeline health

**Mitigation Validation:**
- Large-scale testing of proposed mitigations
- Cost-benefit analysis of different security controls
- Development of security maturity models for DevOps

**Supply Chain Security:**
- Broader investigation of software supply chain attacks
- Research on defending against compromised dependencies
- Analysis of artifact signing and verification effectiveness

---

## Protection Mechanisms

### General Principle: Least Privilege

The common theme across all attacks is privilege escalation. The foundational protection is implementing the **principle of least privilege** throughout the DevOps pipeline:

- Limit account access to only what's necessary
- Shrink the attack surface by reducing potential victims
- Implement strong authentication and authorization
- Regularly audit and review permissions

### Attack-Specific Mitigations

**Protection from Malicious Application Deployment:**
- Implement Kubernetes service accounts tied to specific namespaces
- Use network policies to isolate applications
- Restrict pod deployment to authorized users only
- Monitor for unusual container deployments
- Implement pod security standards to restrict capabilities

**Protection from CI/CD Manipulation:**
- Restrict access to Jenkins with strong RBAC
- Implement code signing and verification
- Use approval workflows for build modifications
- Audit all changes to build configurations
- Isolate Jenkins instances with network segmentation
- Implement build artifact integrity checking

**Protection from Network Exposure:**
- Use Kubernetes admission controllers to review service configurations
- Implement network policies to control traffic flow
- Restrict who can create Ingress/NodePort objects
- Monitor for unauthorized service modifications
- Regular audits of exposed services

**Protection from Namespace Breakout:**
- Restrict hostPath volume usage with Pod Security Standards
- Use PSPs/PSAs to prevent mounting sensitive host paths
- Implement runtime security monitoring (e.g., Falco)
- Restrict privileged container execution
- Regular security audits of pod specifications

---

## Conference Presentation

This research was presented at ICSSP'22 in Pittsburgh, PA.

**Presentation Materials:**
- [View slides](#) *(available on request)*
- [Watch presentation recording](#) *(if available)*
- [Conference announcement post](/research/icssp-2022-conference-announcement)

---

## Access the Paper

**Read the full paper:** https://dl.acm.org/doi/10.1145/3529320.3529325

**Cite this work:**
```
Nicholas Pecka, Lotfi ben Othmane, and Altaz Valani. 2022.
Privilege Escalation Attack Scenarios on the DevOps Pipeline
Within a Kubernetes Environment. In Proceedings of the
International Conference on Software and System Processes and
International Conference on Global Software Engineering
(ICSSP'22), May 20–22, 2022, Pittsburgh, PA, USA.
ACM, New York, NY, USA, 5 pages.
https://doi.org/10.1145/3529320.3529325
```

**BibTeX:**
```bibtex
@inproceedings{pecka2022privilege,
  title={Privilege Escalation Attack Scenarios on the DevOps Pipeline Within a Kubernetes Environment},
  author={Pecka, Nicholas and ben Othmane, Lotfi and Valani, Altaz},
  booktitle={Proceedings of the International Conference on Software and System Processes and International Conference on Global Software Engineering},
  pages={45--49},
  year={2022},
  organization={ACM},
  doi={10.1145/3529320.3529325}
}
```

---

## Discussion

### Reflections on the Research

This research emerged from a critical observation: organizations were adopting DevSecOps tools at an accelerating pace, but security incidents in CI/CD pipelines were also increasing. The disconnect was clear—tool adoption doesn't equal security.

**Key Lessons Learned:**

**1. Complexity is the Enemy of Security**
DevOps pipelines involve many integrated components (code repos, CI/CD, container registries, orchestration platforms). Each integration point is a potential vulnerability.

**2. Automation Amplifies Impact**
The same automation that makes DevOps powerful also means that compromises can affect many systems rapidly. A single backdoor in a CI/CD pipeline can compromise every application it builds.

**3. Default Configurations Are Often Insecure**
Many DevSecOps tools ship with permissive defaults to ease adoption. Without hardening, these defaults create security gaps.

**4. Security Knowledge Gap**
Many DevOps engineers are experts in their tools but may lack deep security knowledge. Bridging this gap through training and secure-by-default configurations is essential.

### Challenges Encountered

**Building a Representative Environment:**
Creating a DevOps pipeline that accurately reflected real-world deployments while remaining manageable for research purposes required careful balance.

**Ethical Considerations:**
Ensuring the research could be published and shared without providing a cookbook for malicious actors. We focused on demonstrating the importance of security rather than creating exploit tools.

**Mitigation Validation:**
Testing that proposed mitigations actually work without introducing operational overhead or breaking legitimate workflows.

---

## Related Work

**Research Papers:**
- Shamim et al. - "Kubernetes Security: Operating Kubernetes Clusters and Applications Safely"
- Minna et al. - "Network Security Issues in Kubernetes"
- Bertuccio - "Software Supply Chain Security Analysis"
- Li et al. - "Cijacking: Cryptomining on CI Platforms"

**Practical Resources:**
- [Kubernetes Goat](https://github.com/madhuakula/kubernetes-goat) - Interactive Kubernetes security playground
- Microsoft Kubernetes Threat Matrix
- NIST Guidelines on Software Supply Chain Security

**On This Site:**
- [Conference Announcement](/research/icssp-2022-conference-announcement)
- More DevSecOps research coming soon

---

## Acknowledgments

**Research Support:**
This research was conducted at Iowa State University under the guidance of Dr. Lotfi ben Othmane. Special thanks to Altaz Valani from Security Compass for industry perspective and collaboration.

**Conference:**
Thank you to the ICSSP'22 program committee for accepting this work and providing valuable feedback during the review process.

**Tools and Infrastructure:**
This research utilized open-source tools including Kubernetes, Jenkins, Docker, GitHub, and Strimzi. Thanks to the maintainers and communities behind these projects.

---

**Questions about this research?** Feel free to reach out via the contact form or leave a comment below.

**Want updates on future research?** Subscribe to get notifications when I publish new research, papers, or conference presentations.

---

**Tags for Ghost:** #research #paper #published #kubernetes #devsecops #security #software-supply-chain
