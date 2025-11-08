# Presenting at CRiSIS 2025: Automated Security Risk Detection Through Call Graph Analysis

I'm excited to share that my latest research on automating threat modeling will be presented at the 20th International Conference on Risks and Security of Internet and Systems (CRiSIS) 2025!

---

## Conference Details

**Conference:** 20th International Conference on Risks and Security of Internet and Systems (CRiSIS 2025)
**Location:** Gatineau, Quebec, Canada
**Dates:** November 5–8, 2025
**Website:** https://crisis2025.uqo.ca/

**My Presentation:**
- **Session:** Security Research Track
- **Format:** Research Paper Presentation
- **Co-authors:** Dr. Lotfi Ben Othmane (University of North Texas) and Dr. Renee Bryce (University of North Texas)

---

## About the Conference

CRiSIS is a prestigious international conference focused on information security, bringing together researchers and practitioners to discuss emerging threats, innovative security techniques, and risk management strategies for Internet systems. Now in its 20th year, CRiSIS has established itself as a leading venue for presenting cutting-edge research in cybersecurity.

This conference is particularly relevant for my research as it emphasizes practical approaches to security challenges in real-world systems, bridging the gap between academic research and industry application.

---

## Paper/Presentation Title

**Title:** Toward Automated Security Risk Detection in Large Software Using Call Graph Analysis

**Co-authors:**
- Nicholas Pecka, University of North Texas & Red Hat
- Dr. Lotfi Ben Othmane, University of North Texas
- Dr. Renee Bryce, University of North Texas

---

## What I'll Be Presenting

### Abstract

Threat modeling plays a critical role in the identification and mitigation of security risks; however, manual approaches are often labor-intensive and prone to error. This paper investigates the automation of software threat modeling through the clustering of call graphs using density-based and community detection algorithms, followed by an analysis of the threats associated with the identified clusters.

The proposed method was evaluated through a case study of the Splunk Forwarder Operator (SFO), wherein selected clustering metrics were applied to the software's call graph to assess pertinent code-density security weaknesses. The results demonstrate the viability of the approach and underscore its potential to facilitate systematic threat assessment.

### Key Topics

I'll be covering:

1. **Automated Threat Modeling Challenges**
   Why manual threat modeling doesn't scale for modern cloud-native applications, and the critical need for automation in continuous deployment environments

2. **Call Graph Clustering Techniques**
   Comparison of four clustering algorithms (DBSCAN, HDBSCAN, Louvain, and Leiden) for analyzing software structure and identifying security-relevant patterns

3. **Heuristic-Based Threat Detection**
   Novel heuristics that map clustering patterns to MITRE CWE categories, enabling automated identification of common security weaknesses in large codebases

4. **Real-World Case Study**
   Analysis of the Splunk Forwarder Operator, a production operator deployed across thousands of Red Hat OpenShift clusters

---

## Why This Work Matters

### The Threat Modeling Problem

Traditional threat modeling is invaluable for security but faces critical challenges:

**The Manual Bottleneck:**
- Requires coordination across multiple stakeholders and domain experts
- No single person can realistically maintain complete models for large systems
- Becomes outdated the moment code changes are deployed
- Often deprioritized in production environments due to complexity

**The Scale Challenge:**
- Modern cloud-native applications evolve continuously
- A single component update can invalidate entire threat models
- Vulnerability scanners miss risks introduced at system-interaction levels
- Traditional tools focus on individual components, not system-wide threats

### Who Benefits from This Research

**For Security Teams:**
- Automate portions of threat modeling that are currently manual
- Maintain up-to-date threat assessments as code evolves
- Identify security risks at scale across large codebases
- Prioritize security reviews based on structural risk indicators

**For Development Teams:**
- Receive automated feedback on architectural security concerns
- Understand security implications of code structure
- Identify potential issues before they reach production
- Reduce dependency on manual security review bottlenecks

**For Organizations:**
- Scale threat modeling to match continuous deployment velocity
- Maintain security visibility across large software portfolios
- Reduce costs associated with manual security assessments
- Enable proactive rather than reactive security approaches

---

## Research Highlights

### Breakthrough Approach

This research introduces a novel automated approach to threat modeling:

**1. Call Graph Generation**
Automatically extract complete call graphs from source code, capturing all function calls and control flow paths.

**2. Intelligent Clustering**
Apply advanced clustering algorithms (HDBSCAN and Leiden) to identify structural patterns in code organization.

**3. Threat Heuristics**
Map clustering patterns to known security weaknesses using MITRE CWE-based heuristics:
- **Bridging Clusters** → CWE-668 (Exposure of Resource to Wrong Sphere)
- **Hotspot Clusters** → CWE-284 (Improper Access Control)
- **Dangling Nodes** → CWE-94 (Code Injection) / CWE-1164 (Irrelevant Code)
- **Hub Nodes** → CWE-20 (Improper Input Validation)
- **Weak Clusters** → CWE-200 (Exposure of Sensitive Information)

### Impressive Results

**Scale Achieved:**
- Analyzed call graph with 39,024 nodes and 286,302 edges
- Over 350,000 lines of call data processed
- Real production software (Splunk Forwarder Operator)

**Algorithm Performance:**
- **HDBSCAN:** 180 clusters in 18.17 seconds, silhouette score 0.5087
- **Leiden:** 366 clusters in 0.26 seconds, modularity score 0.8202
- Both significantly outperformed baseline algorithms (DBSCAN and Louvain)

**Practical Findings:**
- Identified critical hotspot cluster receiving 8,338 calls
- Discovered weak cluster with 687:1 external-to-internal edge ratio
- Flagged overuse of Go's `reflect` package across 23 clusters (potential security concern)

---

## The Journey to This Point

This research emerged from real-world challenges I encountered at Red Hat, where threat modeling large Kubernetes operators proved impractical using traditional manual methods.

### Key Challenges Overcome

**1. Scaling Analysis to Production Code**
Moving from proof-of-concept to analyzing real production software with hundreds of thousands of function calls required significant algorithmic optimization.

**2. Bridging Code Structure and Security**
Developing heuristics that meaningfully connect graph clustering patterns to actual security weaknesses required extensive research into MITRE CWE categories and their relationship to software architecture.

**3. Algorithm Selection and Tuning**
Evaluating four different clustering algorithms to find the right balance between accuracy, speed, and interpretability for security analysis.

**4. Validation in Production Context**
Ensuring findings are actionable for real security teams working on real products, not just theoretically interesting.

---

## What to Expect from the Presentation

The presentation will include:

- **Live Algorithm Demonstration:** Visual walkthrough of clustering algorithms operating on real call graphs
- **Interactive Visualizations:** Graph visualizations showing identified clusters and threat patterns
- **Case Study Deep-Dive:** Detailed analysis of findings from the Splunk Forwarder Operator
- **Practical Application Guide:** How security teams can apply this approach to their own codebases

---

## Real-World Application: Red Hat OpenShift

The case study focuses on the Splunk Forwarder Operator, a component deployed by default across thousands of Red Hat OpenShift clusters worldwide. This represents a real production scenario where:

- The software is critical infrastructure for log collection
- Manual threat modeling is impractical due to continuous updates
- Security impacts thousands of production deployments
- Automated analysis can provide ongoing security visibility

This demonstrates the method's applicability to real-world, large-scale software security challenges.

---

## Looking Forward

I'm hoping to gain from attending the conference:

- **Research Community Feedback:** Validation of our heuristics and methodology from cybersecurity researchers
- **Industry Insights:** Understanding how different organizations approach automated threat modeling
- **Collaboration Opportunities:** Connecting with researchers working on software analysis and security automation
- **Tool Development:** Identifying requirements for turning this research into practical tooling

**Future Research Directions:**
- Expanding heuristics through comprehensive literature review
- Applying approach to additional programming languages beyond Go
- Integration with existing DevSecOps pipelines
- Development of actionable remediation guidance based on findings

---

## Key Contributions

**1. Algorithm Evaluation**
Comprehensive comparison of clustering capabilities across DBSCAN, HDBSCAN, Louvain, and Leiden for security analysis.

**2. Automated Threat Detection Method**
Novel approach applying clustering algorithms to call graphs with security-focused heuristics.

**3. Production Validation**
Evaluation on widely-deployed production software (Splunk Forwarder Operator on Red Hat OpenShift).

---

## Conference Preparation

### Presentation Development

This presentation distills months of research into demonstrating both the technical innovation and practical applicability. The challenge has been making advanced graph theory and clustering algorithms accessible while showing concrete security value.

### Research Reproducibility

All methodology and approaches will be thoroughly documented to enable other researchers and security teams to apply similar techniques to their own software.

---

## Can't Attend? Here's How to Follow Along

- **Paper:** Will be published in conference proceedings
- **Slides:** Will be posted to my website after the presentation
- **Research Materials:** Methodology and approach details available post-conference
- **Follow-up Posts:** Detailed blog posts expanding on findings and practical application

---

## Pre-Conference Resources

**Recommended Background:**

For those interested in the technical foundations:
- Graph Theory basics (nodes, edges, clustering)
- Call graph concepts in static analysis
- MITRE CWE (Common Weakness Enumeration) framework
- Threat modeling fundamentals (STRIDE, attack trees)

**Related Topics:**
- Static application security testing (SAST)
- Software architecture visualization
- DevSecOps automation
- Cloud-native security

---

## Questions?

Have questions about automated threat modeling? Interested in how this could apply to your software? Feel free to reach out or leave a comment below!

---

**Following my research?** Subscribe to get notifications when I publish new research, papers, or conference presentations.

**Will you be at CRiSIS 2025?** Let me know—I'd love to connect!

---

*This post will be updated after the conference with presentation materials and reflections.*
