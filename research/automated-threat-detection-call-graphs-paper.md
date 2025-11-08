# Toward Automated Security Risk Detection in Large Software Using Call Graph Analysis

Published research demonstrating how call graph clustering and heuristic analysis can automate threat modeling for large-scale cloud-native applications, addressing the scalability challenges of manual security assessment.

---

## Paper Information

**Title:** Toward Automated Security Risk Detection in Large Software Using Call Graph Analysis

**Authors:** Nicholas Pecka (University of North Texas & Red Hat), Lotfi Ben Othmane (University of North Texas), Renee Bryce (University of North Texas)

**Published in:** Proceedings of the 20th International Conference on Risks and Security of Internet and Systems (CRiSIS 2025)

**Publication Date:** November 5–8, 2025

**Status:** Published

**Conference Location:** Gatineau, Quebec, Canada

---

## Abstract

Threat modeling plays a critical role in the identification and mitigation of security risks; however, manual approaches are often labor-intensive and prone to error. This paper investigates the automation of software threat modeling through the clustering of call graphs using density-based and community detection algorithms, followed by an analysis of the threats associated with the identified clusters.

The proposed method was evaluated through a case study of the Splunk Forwarder Operator (SFO), wherein selected clustering metrics were applied to the software's call graph to assess pertinent code-density security weaknesses. The results demonstrate the viability of the approach and underscore its potential to facilitate systematic threat assessment.

**This work contributes to the advancement of scalable, semi-automated threat modeling frameworks tailored for modern cloud-native environments.**

---

## Research Context

### The Problem

Modern software development faces a critical security challenge: **threat modeling doesn't scale**.

**Why Traditional Threat Modeling Falls Short:**

1. **Labor-Intensive Manual Process**
   - Requires coordination across multiple stakeholders and domain experts
   - No single person can maintain complete models for large, complex systems
   - Becomes a bottleneck in continuous deployment pipelines

2. **Constantly Outdated**
   - Software systems evolve continuously with every deployment
   - Initial threat models become obsolete immediately after code changes
   - Single component updates can invalidate entire threat models

3. **Ignored After Deployment**
   - Typically conducted only during design phase
   - Deprioritized in production due to manual effort required
   - Creates growing security debt as systems age

4. **Incomplete Coverage**
   - Vulnerability scanners identify component-level issues
   - Miss risks introduced at system-interaction levels
   - Rarely capture how applications interact with deployed infrastructure

**The Core Challenge:**
How do we maintain comprehensive, up-to-date threat models for large, continuously-evolving cloud-native applications without requiring massive manual effort?

### Why It Matters

**For the Security Industry:**
As organizations adopt DevSecOps and continuous deployment, the gap between deployment velocity and security assessment capability widens dangerously. Automated threat modeling is essential to close this gap.

**For Cloud-Native Environments:**
Kubernetes operators, microservices, and cloud-native applications introduce complexity that manual threat modeling cannot address at scale. These systems require new approaches.

**For Software Supply Chain Security:**
Recent high-profile supply chain attacks demonstrate that understanding system-wide security implications—not just individual component vulnerabilities—is critical.

---

## Key Contributions

This research makes the following contributions:

1. **Comprehensive Algorithm Evaluation**
   Evaluation and comparison of four clustering algorithms' capabilities for security-focused call graph analysis: DBSCAN, HDBSCAN, Louvain, and Leiden

2. **Novel Automated Threat Detection Approach**
   Proposed methodology applying clustering-based algorithms to call graphs with heuristics for threat identification based on MITRE CWE categories

3. **Production Software Validation**
   Evaluation on the Splunk Forwarder Operator, a widely-used logging agent in Red Hat OpenShift environments, demonstrating real-world applicability

4. **Security-Focused Heuristics**
   Development of five heuristics mapping structural code patterns to specific Common Weakness Enumerations (CWEs)

---

## Methodology Overview

### Approach

Our research employs a multi-phase methodology combining static analysis, graph clustering, and security heuristics:

**Phase 1: Call Graph Generation**
Generate complete call graphs from source code using static analysis tools, capturing all function calls, control flow, and code structure.

**Phase 2: Clustering Algorithm Application**
Apply both density-based (HDBSCAN) and graph-based (Leiden) clustering algorithms to identify structural patterns and modular components.

**Phase 3: Heuristic-Based Analysis**
Analyze clustering results using security-focused heuristics that map structural patterns to known security weaknesses (MITRE CWEs).

**Phase 4: Threat Identification**
Generate prioritized list of potential security risks based on heuristic findings for manual validation and remediation.

### Why Call Graphs?

Call graphs are particularly valuable for security analysis because they:

- **Capture Entry Points:** Show all paths attackers might use to reach sensitive operations
- **Reveal Data Flows:** Illustrate how data moves through the system
- **Identify Sensitive Interactions:** Highlight privileged operations and trust boundaries
- **Expose Legacy Code:** Reveal unused or forgotten functions that may present hidden risks
- **Enable Verification:** Allow developers to confirm necessity of code paths
- **Auto-Update:** Generated from code, automatically reflect current implementation

### Tools and Technologies

**Call Graph Generation:**
- **go-callvis:** Static analysis tool for generating call graphs from Go code
- Generated from `main.go` entry point to capture complete control flow

**Clustering Algorithms Evaluated:**

1. **DBSCAN (Density-Based Spatial Clustering of Applications with Noise)**
   - Baseline density-based clustering algorithm
   - Uses fixed ε (epsilon) parameter for density threshold
   - Limitation: Struggles with non-uniform densities

2. **HDBSCAN (Hierarchical DBSCAN)**
   - Advanced density-based clustering
   - Handles variable densities through hierarchy
   - Uses mutual reachability distance for robustness

3. **Louvain**
   - Community detection algorithm
   - Optimizes modularity metric
   - Limitation: Can produce inconsistent partitions

4. **Leiden**
   - Improved community detection
   - Adds refinement phase ensuring connected communities
   - Guarantees stability and convergence

**Case Study Software:**
- **Splunk Forwarder Operator (SFO):** Red Hat OpenShift operator for log collection
- **Scale:** 39,024 nodes, 286,302 edges, 350,000+ lines of call data
- **Language:** Go
- **Deployment:** Production software across thousands of clusters

**Evaluation Metrics:**
- **Silhouette Score:** For density-based clustering quality (DBSCAN, HDBSCAN)
- **Modularity:** For graph-based clustering quality (Louvain, Leiden)
- **Runtime Performance:** Algorithm execution time
- **Cluster Quality:** Distribution and characteristics of identified clusters

---

## Key Findings

### Finding 1: HDBSCAN Significantly Outperforms DBSCAN for Call Graph Analysis

**Comparison Results:**

**DBSCAN Performance:**
- Best configuration: eps=0.09, minimum samples=5
- Generated: 32 clusters
- Runtime: 2.64 seconds
- Silhouette score: 0.0227 (near-random clustering)
- **Conclusion:** Inadequate for call graph analysis at scale

**HDBSCAN Performance:**
- Configuration: minimum cluster size=8
- Generated: 180 clusters
- Runtime: 18.17 seconds
- Silhouette score: 0.5087 (meaningful structure)
- **Conclusion:** Effective at identifying dense, interconnected components

**Implication:**
Variable-density clustering is essential for call graphs, where different modules naturally have different connectivity patterns. HDBSCAN's hierarchical approach successfully identifies both tightly-coupled core modules and looser peripheral components.

### Finding 2: Leiden Algorithm Provides Optimal Balance of Quality and Performance

**Comparison Results:**

**Louvain Performance:**
- Generated: 367 clusters
- Runtime: 6.45 seconds
- Modularity: 0.8090
- **Issue:** Inconsistent partition quality

**Leiden Performance:**
- Generated: 366 clusters
- Runtime: 0.26 seconds (24x faster than Louvain!)
- Modularity: 0.8202 (higher quality)
- **Advantage:** Refinement phase ensures connected communities

**Implication:**
Leiden's combination of high modularity, fast execution, and stable results makes it ideal for iterative threat modeling workflows and integration into CI/CD pipelines. The sub-second execution time enables real-time analysis.

### Finding 3: Heuristic Analysis Successfully Identifies Security-Relevant Patterns

**Five Security Heuristics Developed:**

**1. Bridging Clusters (CWE-668: Exposure of Resource to Wrong Sphere)**
- **Pattern:** Small clusters with many external connections
- **Security Implication:** Acts as bridge between trust boundaries
- **SFO Results:** 0 instances (HDBSCAN), 0 instances (Leiden)

**2. Hotspot Clusters (CWE-284: Improper Access Control)**
- **Pattern:** Large clusters with high incoming call volume
- **Security Implication:** Central services that may lack proper access control
- **SFO Results:** 162 instances (HDBSCAN), 24 instances (Leiden)
- **Critical Finding:** Cluster 179 received 8,338 calls (significantly disproportionate)

**3. Dangling Nodes (CWE-94: Code Injection / CWE-1164: Irrelevant Code)**
- **Pattern:** Nodes connected to only one node in different cluster
- **Security Implication:** Loosely controlled logic or abandoned code
- **SFO Results:** 57 instances (HDBSCAN), 485 instances (Leiden)

**4. Hub Nodes (CWE-20: Improper Input Validation)**
- **Pattern:** Individual nodes with unusually high connection count
- **Security Implication:** Aggregation/parsing points that may lack validation
- **SFO Results:** 80 instances (HDBSCAN), 217 instances (Leiden)
- **Critical Finding:** `(reflect.Value).Call` appeared across 23 clusters

**5. Weak Clusters (CWE-200: Exposure of Sensitive Information)**
- **Pattern:** Clusters with more external than internal connections
- **Security Implication:** Poor encapsulation exposing internal data
- **SFO Results:** 27 instances (HDBSCAN), 0 instances (Leiden)
- **Critical Finding:** Cluster 152 had 687:1 external-to-internal edge ratio

**Implication:**
Structural patterns in code organization correlate with known security weaknesses. Automated detection of these patterns enables proactive security assessment at scale.

### Finding 4: Reflect Package Overuse Indicates Potential Security Concern

**Key Discovery:**
The Go `reflect` package function `(reflect.Value).Call` was identified as:
- Top candidate in both dangling node and hub node heuristics
- Connection split: 1,968 outgoing calls to different clusters (vs. 64 for second-ranked)
- Present as hub across 23 different clusters
- Second-ranked hub appeared in only 9 clusters

**Security Implications:**

**Why Reflection is Risky:**
- Enables runtime type manipulation
- Bypasses compile-time type safety
- Can obscure data flow and control flow
- Makes static analysis more difficult
- May indicate overly dynamic, hard-to-verify code paths

**Recommendation:**
Excessive use of reflection across many modules suggests poor cohesion and potential architectural concerns requiring security review.

**Implication:**
Language-specific security anti-patterns can be automatically detected through call graph analysis, providing language-aware security guidance.

---

## Practical Applications

### For Security Engineers

**Immediate Actions:**

1. **Integrate into CI/CD Pipeline**
   - Run call graph analysis on every build
   - Generate automated reports flagging high-risk clusters
   - Block deployments with critical security patterns

2. **Prioritize Manual Reviews**
   - Focus on clusters flagged by multiple heuristics
   - Review hotspot clusters receiving disproportionate traffic
   - Investigate weak clusters with poor encapsulation

3. **Track Security Debt**
   - Monitor trends in heuristic findings over time
   - Measure improvement or degradation of code structure
   - Set thresholds for acceptable security patterns

**Long-Term Strategy:**

- Develop custom heuristics for organization-specific patterns
- Build security metrics based on call graph characteristics
- Create continuous threat modeling dashboards
- Enable shift-left security through automated early detection

### For Development Teams

**Architectural Guidance:**

1. **Code Organization**
   - Minimize external connections for sensitive modules
   - Ensure proper encapsulation (high internal coherence)
   - Avoid creating bottleneck clusters (hotspots)
   - Clean up dangling nodes and orphaned code

2. **Language-Specific Best Practices**
   - Limit reflection usage to necessary cases
   - Prefer compile-time type safety over runtime flexibility
   - Maintain clear module boundaries
   - Document cross-module dependencies

**Integration Points:**

- IDE plugins showing real-time security warnings
- Pre-commit hooks preventing introduction of security anti-patterns
- Code review automation highlighting structural concerns
- Architecture documentation auto-generated from call graphs

### For Organizations

**Strategic Benefits:**

**Scalability:**
- Automated analysis handles codebases of any size
- Sub-second execution enables continuous assessment
- Reduces dependency on scarce security expertise

**Cost Reduction:**
- Decreases manual security review time
- Identifies issues earlier (cheaper to fix)
- Prevents security incidents through proactive detection

**Compliance:**
- Provides audit trail of security assessments
- Demonstrates due diligence in threat identification
- Maps findings to industry-standard CWE categories

**Velocity:**
- Doesn't slow down development cycles
- Provides immediate feedback
- Enables secure continuous deployment

---

## Detailed Heuristics and CWE Mappings

### Bridging Clusters → CWE-668: Exposure of Resource to Wrong Sphere

**Pattern Detection:**
Identify small clusters (< N nodes) with high external connection ratio (> M% outgoing connections).

**Why It Matters:**
Clusters acting as bridges between larger modules may cross trust boundaries, potentially allowing unauthorized resource access across security domains.

**Example Scenario:**
A small utility module that facilitates communication between a public API and an internal database, potentially bypassing access controls.

**Recommended Actions:**
- Review access control mechanisms at bridge points
- Verify proper authentication/authorization
- Consider architectural refactoring to eliminate bridge
- Add security monitoring at boundary crossings

### Hotspot Clusters → CWE-284: Improper Access Control

**Pattern Detection:**
Identify clusters receiving significantly more incoming calls than average (> X standard deviations above mean).

**Why It Matters:**
Clusters processing high volumes of requests are attractive targets for attack and may have overlooked access control vulnerabilities due to complexity.

**Example Scenario:**
A central API handler receiving thousands of calls from various modules may have permissive access controls due to diverse legitimate use cases.

**Recommended Actions:**
- Implement robust input validation
- Apply principle of least privilege
- Add rate limiting and abuse protection
- Conduct focused penetration testing on hotspots

### Dangling Nodes → CWE-94 / CWE-1164

**Pattern Detection:**
Identify nodes with:
- Only one connection to another cluster
- No connections within own cluster
- Potentially orphaned or misplaced code

**Why It Matters:**
Loosely connected code may represent:
- Abandoned features not properly removed
- Potential injection points
- Dead code that still executes

**Example Scenario:**
An old logging function still called from one location that writes unsanitized data to files.

**Recommended Actions:**
- Remove truly dead code
- Refactor misplaced code to proper module
- Add proper input validation if code is needed
- Document and monitor if intentionally isolated

### Hub Nodes → CWE-20: Improper Input Validation

**Pattern Detection:**
Identify individual nodes with:
- Unusually high incoming or outgoing connection count
- Connections across many different modules/clusters

**Why It Matters:**
Hub nodes often perform aggregation, routing, or parsing of data from multiple sources and may lack comprehensive input validation.

**Example Scenario:**
A central request router accepting data from dozens of different sources without proper sanitization.

**Recommended Actions:**
- Implement comprehensive input validation
- Add schema verification for all inputs
- Use allow-listing rather than deny-listing
- Consider breaking up overly centralized hubs

### Weak Clusters → CWE-200: Exposure of Sensitive Information

**Pattern Detection:**
Identify clusters with:
- More external connections than internal connections
- Low cohesion (ratio < threshold)
- Poor module encapsulation

**Why It Matters:**
Modules with more external than internal connections likely expose internal implementation details and may leak sensitive information.

**Example Scenario:**
A database access module that exposes individual query functions rather than providing a clean API boundary.

**Recommended Actions:**
- Refactor to improve encapsulation
- Create clear API boundaries
- Hide internal implementation details
- Review what data crosses module boundaries

---

## Algorithm Performance Comparison

### Computational Efficiency

**Runtime Comparison (39,024 nodes, 286,302 edges):**
- **DBSCAN:** 2.64 seconds
- **Louvain:** 6.45 seconds
- **HDBSCAN:** 18.17 seconds
- **Leiden:** 0.26 seconds ✓ (fastest)

**Quality vs. Speed Trade-offs:**

**For Real-Time CI/CD Integration:**
- **Leiden** is ideal: sub-second execution, high modularity
- Can run on every commit without slowing pipelines

**For Deep Security Analysis:**
- **HDBSCAN** provides complementary insights to Leiden
- Worth the additional runtime for comprehensive assessment
- Run nightly or weekly for detailed reports

### Scalability Considerations

**Tested Scale:**
- Successfully analyzed 350,000+ lines of call data
- 39,024 nodes, 286,302 edges
- Representative of large production applications

**Expected Scalability:**
- Leiden's algorithm complexity suggests linear scalability
- HDBSCAN may require optimization for very large graphs (>100k nodes)
- Parallel processing opportunities exist for independent cluster analysis

**Practical Limits:**
- Most applications will fall within tested scale
- For extreme cases (millions of nodes), consider:
  - Analyzing subsystems independently
  - Incremental analysis of code changes only
  - Distributed graph processing

---

## Limitations and Future Work

### Current Limitations

**1. Language Coverage**
- Current implementation focuses on Go
- Call graph generation differs across languages
- Heuristics may need language-specific tuning

**2. Heuristic Completeness**
- Five initial heuristics developed
- Many additional CWE categories could be mapped
- Comprehensive literature review needed

**3. False Positive Rate**
- Not all flagged patterns represent actual vulnerabilities
- Manual validation still required
- Need for precision/recall measurements

**4. Dynamic Behavior**
- Static call graphs miss runtime-only paths
- Reflection and dynamic dispatch complicate analysis
- Integration with dynamic analysis needed

**5. Remediation Guidance**
- Current approach identifies issues
- Limited guidance on how to fix identified problems
- Need for actionable remediation recommendations

### Future Research Directions

**1. Multi-Language Support**
- Extend to Python, Java, JavaScript, C++
- Language-agnostic call graph representation
- Language-specific security heuristics

**2. Comprehensive Heuristic Development**
- Literature review mapping clustering patterns to CWEs
- Machine learning to discover new security-relevant patterns
- Validation across diverse codebases

**3. Integration with Existing Tools**
- Combine with SAST/DAST findings
- Integrate with vulnerability databases
- Export to threat modeling tools (e.g., Microsoft Threat Modeling Tool)

**4. Remediation Automation**
- Suggest architectural refactoring
- Generate secure code patterns
- Provide example fixes for common issues

**5. Continuous Threat Modeling**
- Track changes over time
- Identify security regression
- Measure security improvement metrics

**6. Industry Validation**
- Evaluate across multiple organizations
- Measure false positive/negative rates
- Compare with manual threat modeling results

**7. Tool Development**
- Open-source implementation
- IDE integrations
- CI/CD plugins
- Web-based visualization dashboard

---

## Case Study Deep Dive: Splunk Forwarder Operator

### Why This Case Study Matters

**Production Relevance:**
- Deployed by default on Red Hat OpenShift clusters
- Used by thousands of organizations worldwide
- Critical for log collection and observability
- Represents real security stakes

**Complexity:**
- Large enough to demonstrate scalability (39k+ nodes)
- Complex enough to reveal meaningful patterns
- Production code quality (not toy example)

**Representative:**
- Typical Kubernetes operator architecture
- Common Go programming patterns
- Real-world deployment scenarios

### Key Findings Summary

**Hotspot Analysis (CWE-284):**
- Cluster 179: 8,338 incoming calls
- Cluster 98: 6,806 incoming calls
- Significant disparity suggests access control review needed
- Recommendation: Implement rate limiting and enhanced authorization

**Weak Cluster Analysis (CWE-200):**
- Cluster 152: 458 nodes, 687:1 external-to-internal edge ratio
- Next highest: 24 nodes, 120:1 ratio
- Dramatic outlier indicates serious encapsulation issue
- Recommendation: Major refactoring to improve module boundaries

**Reflection Pattern Analysis (CWE-20):**
- `(reflect.Value).Call`: 1,968/3,447 cross-cluster connections
- Appears as hub across 23 clusters
- Significant overuse compared to norms
- Recommendation: Reduce reflection, improve type safety

### Lessons Learned

1. **Automated Analysis is Feasible:** Successfully identified actionable security concerns in production code
2. **Patterns Emerge Clearly:** Outliers and concerning patterns easily identified in clustering results
3. **Context Matters:** Understanding the software's purpose essential for interpreting findings
4. **Manual Validation Required:** Automated findings require expert review to determine actual risk
5. **Continuous Value:** Re-running analysis after changes shows security improvement over time

---

## Implementation Guide

### For Teams Wanting to Apply This Approach

**Step 1: Generate Call Graphs**
```bash
# For Go projects
go-callvis -format=dot -file=callgraph ./path/to/main.go
```

**Step 2: Parse and Load Graph Data**
- Convert DOT format to graph representation
- Load into graph analysis library (NetworkX, igraph, etc.)

**Step 3: Apply Clustering**
```python
# Example using HDBSCAN
import hdbscan
clusterer = hdbscan.HDBSCAN(min_cluster_size=8)
cluster_labels = clusterer.fit_predict(graph_data)

# Example using Leiden
import leidenalg
import igraph as ig
partition = leidenalg.find_partition(graph, leidenalg.ModularityVertexPartition)
```

**Step 4: Apply Heuristics**
- Calculate metrics for each cluster
- Apply threshold-based detection
- Rank findings by severity

**Step 5: Generate Reports**
- List flagged clusters with details
- Provide CWE mappings
- Include visualization of concerning areas

**Step 6: Manual Review**
- Security team reviews flagged areas
- Validates actual security impact
- Plans remediation

### Integration Points

**CI/CD Pipeline:**
```yaml
# Example GitLab CI job
security-callgraph-analysis:
  stage: security
  script:
    - generate-callgraph.sh
    - run-clustering-analysis.sh
    - check-thresholds.sh
  artifacts:
    reports:
      security: callgraph-security-report.json
```

**Pre-Commit Hook:**
- Analyze only changed files
- Fast feedback for developers
- Block commits introducing critical patterns

**Nightly Security Scan:**
- Full analysis of entire codebase
- Trending reports over time
- Email reports to security team

---

## Conference Presentation

This research was presented at CRiSIS 2025 in Gatineau, Quebec, Canada.

**Presentation Materials:**
- [View slides](#) *(available on request)*
- [Conference announcement post](/research/crisis-2025-conference-announcement)

---

## Access the Paper

**Read the full paper:** *(Conference proceedings link to be added)*

**Cite this work:**
```
Nicholas Pecka, Lotfi Ben Othmane, and Renee Bryce. 2025.
Toward Automated Security Risk Detection in Large Software Using
Call Graph Analysis. In Proceedings of the 20th International
Conference on Risks and Security of Internet and Systems
(CRiSIS 2025), November 5–8, 2025, Gatineau, Quebec, Canada.
```

**BibTeX:**
```bibtex
@inproceedings{pecka2025automated,
  title={Toward Automated Security Risk Detection in Large Software Using Call Graph Analysis},
  author={Pecka, Nicholas and Ben Othmane, Lotfi and Bryce, Renee},
  booktitle={Proceedings of the 20th International Conference on Risks and Security of Internet and Systems},
  year={2025},
  organization={CRiSIS}
}
```

---

## Discussion

### Reflections on the Research

This work represents a significant step toward making threat modeling practical for continuous deployment environments. The journey from concept to validated approach revealed several important insights.

**Surprises:**

1. **Algorithm Performance Gap:** The dramatic difference between Leiden (0.26s) and other algorithms was unexpected and game-changing for practical deployment.

2. **Reflection Pattern:** The extent of `reflect` package usage was striking and demonstrates how automated analysis can reveal architectural concerns humans might miss.

3. **Heuristic Effectiveness:** The clarity with which security-relevant patterns emerged from clustering results exceeded initial expectations.

**Challenges:**

1. **Tuning Parameters:** Finding optimal clustering parameters required extensive experimentation. Future work should explore auto-tuning approaches.

2. **Ground Truth Validation:** Without pre-existing threat models for comparison, validating results required significant manual security review.

3. **Language Specifics:** Go's reflection mechanisms created unique patterns that may not appear in other languages, highlighting need for language-specific heuristics.

**Next Steps:**

The most important outcome is demonstrating feasibility. The next challenge is moving from research prototype to production-ready tooling that security teams can deploy in their environments.

---

## Related Work

**Graph-Based Security Analysis:**
- Herranz-Oliveros et al. - "DBSCAN and HDBSCAN for Lateral Movement Detection in Networks"
- Gulbay and Demirici - "Leiden Algorithm for APT Analysis"

**Threat Modeling:**
- Microsoft Threat Modeling Tool
- STRIDE methodology
- PASTA framework

**Static Analysis:**
- Traditional SAST tools (SonarQube, Checkmarx, Fortify)
- Differences from component-level vulnerability detection

**Software Architecture:**
- Call graph analysis for software understanding
- Modularity metrics and software quality

**On This Site:**
- [Conference Announcement](/research/crisis-2025-conference-announcement)
- [Previous DevOps Security Research](/research/icssp-2022-conference-announcement)

---

## Acknowledgments

**Research Support:**
This research was conducted at the University of North Texas under the guidance of Dr. Lotfi Ben Othmane and Dr. Renee Bryce.

**Industry Partnership:**
Special thanks to Red Hat for supporting this research and providing the real-world context of the Splunk Forwarder Operator case study.

**Conference:**
Thank you to the CRiSIS 2025 program committee for accepting this work and providing valuable feedback.

**Tools:**
This research utilized open-source tools including go-callvis, HDBSCAN, Leiden algorithm, and various graph analysis libraries. Thanks to the maintainers and contributors.

---

**Questions about this research?** Interested in collaborating on automated threat modeling? Feel free to reach out via the contact form or leave a comment below.

**Want updates on future research?** Subscribe to get notifications when I publish new research, papers, or conference presentations.

---

**Tags for Ghost:** #research #paper #threat-modeling #call-graphs #kubernetes #security-automation #hdbscan #leiden-algorithm
