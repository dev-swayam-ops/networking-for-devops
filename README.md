# 🚀 Networking for DevOps

A comprehensive, hands-on learning resource for mastering networking fundamentals from a **DevOps and cloud perspective**. This course covers everything from networking basics to advanced troubleshooting, with real-world scenarios and practical labs.

---

## 📚 Course Overview

This 13-module course progresses from fundamental networking concepts to advanced cloud networking and troubleshooting. Each module includes:

- 📖 **Detailed explanations** of key concepts
- 🏋️ **Hands-on labs** with step-by-step instructions
- 📝 **Practical exercises** (easy and medium difficulty)
- ✅ **Complete solutions** with command examples
- 📋 **Command cheatsheets** for quick reference

### Course Structure

| Module | Topic | Focus Area | Level |
|--------|-------|-----------|-------|
| **00** | Setup & Networking Tools | Environment preparation | Beginner |
| **01** | Networking Fundamentals | Basic concepts and protocols | Beginner |
| **02** | OSI & TCP/IP Models | Network architecture | Beginner |
| **03** | IP Addressing & Subnetting | IP addressing schemes | Beginner |
| **04** | CIDR & Route Tables | Routing and CIDR notation | Intermediate |
| **05** | DNS Fundamentals | Domain name resolution | Beginner |
| **06** | HTTP/HTTPS & TLS | Web protocols and encryption | Intermediate |
| **07** | Load Balancing & Proxies | Traffic distribution | Intermediate |
| **08** | Firewalls, NACLs & Security Groups | Network security | Intermediate |
| **09** | Cloud Networking (AWS VPC) | AWS VPC and networking | Intermediate |
| **10** | VPN, Direct Connect & Peering | Cloud connectivity | Advanced |
| **11** | Kubernetes/EKS Networking | Container networking | Advanced |
| **12** | Network Security & Zero Trust | Advanced security | Advanced |
| **13** | Troubleshooting & Debugging | Diagnostics and incident response | Advanced |

---

## 🎯 Learning Objectives

By completing this course, you will:

✅ Understand networking fundamentals and the OSI model  
✅ Master IP addressing, subnetting, and CIDR notation  
✅ Learn DNS resolution and how services communicate  
✅ Secure networks with firewalls, NACLs, and security groups  
✅ Design and manage VPCs in AWS  
✅ Set up secure connectivity with VPNs and Direct Connect  
✅ Deploy and troubleshoot Kubernetes networking  
✅ Implement zero-trust security models  
✅ Diagnose and fix network issues using industry tools  
✅ Build robust incident response procedures  

---

## 🏃 Getting Started

### Prerequisites
- Basic Linux/Unix command line knowledge
- Docker installed (for some exercises)
- kubectl installed (for Kubernetes modules)
- AWS CLI configured (for AWS modules)
- Text editor or IDE (VS Code recommended)

### How to Use This Repository

#### Option 1: Sequential Learning (Recommended)
Start from Module 00 and progress through each module:

```bash
# Navigate to a module
cd 00-setup-and-networking-tools

# Read the README for concepts
cat README.md

# Work through exercises
cat exercises.md

# Check your solutions
cat solutions.md

# Quick command reference
cat cheatsheet.md
```

#### Option 2: Topic-Based Learning
Jump to the module that interests you:

```bash
# Learn about Kubernetes networking
cd 11-kubernetes-eks-networking
cat README.md
```

#### Option 3: Quick Reference
Use cheatsheets for fast command lookups:

```bash
# Get DNS troubleshooting commands
cat 05-dns-fundamentals/cheatsheet.md

# Get Kubernetes diagnostics commands
cat 11-kubernetes-eks-networking/cheatsheet.md
```

---

## 📖 Module Details

### **Module 00: Setup and Networking Tools**
- Installing and configuring networking tools
- Essential utilities: ping, traceroute, netstat, tcpdump, curl
- Understanding network tool output
- [→ Start Module 00](./00-setup-and-networking-tools/)

### **Module 01: Networking Fundamentals**
- Protocols: TCP, UDP, IP, ICMP
- Packets and frames
- Port numbers and services
- [→ Start Module 01](./01-networking-fundamentals/)

### **Module 02: OSI and TCP/IP Models**
- Seven-layer OSI model
- TCP/IP model comparison
- Protocol stack understanding
- [→ Start Module 02](./02-osi-and-tcp-ip-models/)

### **Module 03: IP Addressing and Subnetting**
- IPv4 and IPv6 addressing
- Subnet masks and VLSM
- Address allocation strategies
- [→ Start Module 03](./03-ip-addressing-and-subnetting/)

### **Module 04: CIDR and Route Tables**
- CIDR notation and benefits
- Routing decisions
- Route table configuration
- [→ Start Module 04](./04-cidr-and-route-tables/)

### **Module 05: DNS Fundamentals**
- DNS resolution process
- Record types (A, AAAA, CNAME, MX, NS)
- CoreDNS and Kubernetes DNS
- [→ Start Module 05](./05-dns-fundamentals/)

### **Module 06: HTTP/HTTPS and TLS**
- HTTP/2 and HTTP/3
- TLS handshake and certificates
- HTTPS setup and validation
- [→ Start Module 06](./06-http-https-and-tls/)

### **Module 07: Load Balancing and Proxies**
- Load balancing algorithms
- Reverse proxies
- AWS ELB, ALB, NLB
- [→ Start Module 07](./07-load-balancing-and-proxies/)

### **Module 08: Firewalls, NACLs, and Security Groups**
- AWS Security Groups
- Network ACLs
- Stateful vs stateless filtering
- [→ Start Module 08](./08-firewalls-nacl-security-groups/)

### **Module 09: Cloud Networking (AWS VPC)**
- VPC design and architecture
- Subnets and routing
- IGW, NAT, VPC endpoints
- [→ Start Module 09](./09-cloud-networking-aws-vpc/)

### **Module 10: VPN, Direct Connect, and Peering**
- Site-to-Site VPN
- AWS Direct Connect
- VPC Peering and Transit Gateway
- [→ Start Module 10](./10-connectivity-vpn-direct-connect-peering/)

### **Module 11: Kubernetes/EKS Networking**
- Pod networking and CNI
- Service discovery
- Network policies
- [→ Start Module 11](./11-kubernetes-eks-networking/)

### **Module 12: Network Security and Zero Trust**
- Zero-trust architecture
- Microsegmentation
- WAF and DDoS protection
- [→ Start Module 12](./12-network-security-and-zero-trust/)

### **Module 13: Troubleshooting and Debugging**
- Systematic troubleshooting methodology
- Packet capture and analysis
- Log analysis and RCA
- Incident response procedures
- [→ Start Module 13](./13-troubleshooting-and-debugging/)

---

## 📁 File Structure for Each Module

Each module directory contains standardized files:

```
module-name/
├── README.md          # Comprehensive guide with concepts and hands-on lab
├── exercises.md       # 10 exercises (5 easy, 5 medium difficulty)
├── solutions.md       # Complete solutions with explanations
└── cheatsheet.md      # Command reference and quick syntax
```

### File Descriptions

**README.md**
- Learning objectives
- Key concepts explained
- 8-10 step hands-on lab with real-world scenarios
- Common mistakes to avoid
- Troubleshooting scenarios

**exercises.md**
- 5 easy exercises (conceptual understanding)
- 5 medium exercises (practical application)
- Progressive difficulty
- Real-world scenarios

**solutions.md**
- Complete answers to all exercises
- Command examples and expected outputs
- Explanation of why each solution works
- Best practices and tips

**cheatsheet.md**
- Quick command reference tables
- Syntax examples
- Common use cases
- Copy-paste ready commands

---

## 🛠️ Tools and Technologies Covered

### Networking Tools
- `ping`, `traceroute`, `netstat`, `ss`, `nslookup`, `dig`, `curl`, `wget`
- `tcpdump` (packet capture and analysis)
- `iptables` (firewall rules)

### Cloud Platforms
- **AWS**: VPC, Security Groups, NACLs, Load Balancers, Route 53, VPN, Direct Connect
- **Kubernetes**: Pods, Services, Network Policies, Ingress

### Container Technologies
- **Docker**: Container networking, networks
- **Kubernetes/EKS**: Pod networking, CNI, service discovery

### Monitoring and Debugging
- CloudWatch (AWS monitoring)
- VPC Flow Logs (network traffic analysis)
- tcpdump and Wireshark (packet analysis)
- kubectl (Kubernetes diagnostics)

---

## 🎓 Recommended Learning Path

### For Complete Beginners
```
00 → 01 → 02 → 03 → 04 → 05 → 06 → 07 → 08 → 13
```
(Skip cloud-specific modules initially)

### For DevOps Engineers
```
00 → 01 → 02 → 03 → 04 → 05 → 06 → 07 → 08 → 09 → 10 → 13
```
(Focus on practical cloud networking)

### For Kubernetes Specialists
```
00 → 05 → 11 → 12 → 13
```
(Fast-track to Kubernetes networking)

### For Security Engineers
```
00 → 01 → 02 → 08 → 12 → 13
```
(Focus on security and troubleshooting)

---

## 💡 How to Succeed with This Course

### 1. **Read Actively**
- Don't just read the README; take notes
- Try to predict answers before checking solutions

### 2. **Hands-On Practice**
- Run every command shown in the labs
- Modify commands and see what happens
- Break things intentionally to learn

### 3. **Complete All Exercises**
- Start with easy exercises to build confidence
- Medium exercises prepare you for real-world scenarios
- Check solutions only after attempting

### 4. **Use Cheatsheets**
- Reference during labs and exercises
- Create your own version with notes
- Print key sections for quick lookup

### 5. **Build Mental Models**
- Draw network diagrams
- Trace packets through layers
- Connect concepts to your work environment

---

## 🔧 Setup Instructions

### Install Required Tools (Linux/Mac)
```bash
# Update package manager
sudo apt-get update  # Debian/Ubuntu
# or
brew update          # macOS

# Install essential tools
sudo apt-get install -y \
  net-tools \
  iputils-ping \
  traceroute \
  curl \
  wget \
  dnsutils \
  tcpdump \
  netcat-openbsd

# Install Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Install kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# Install AWS CLI
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

### Clone This Repository
```bash
git clone https://github.com/your-username/networking-for-devops.git
cd networking-for-devops
```

---

## 📊 Course Statistics

- **13 Modules** covering networking fundamentals to advanced topics
- **52 Files** (4 files per module × 13 modules)
- **10,000+ Lines** of comprehensive learning material
- **130+ Exercises** with complete solutions
- **500+ Commands** with examples and explanations

---

## 🤝 Contributing

Contributions are welcome! To improve this course:

1. Found a typo or error? Open an issue
2. Have a better explanation? Submit a pull request
3. Know a useful tool or command? Add it to a cheatsheet
4. Completed the course? Share your feedback

---

## 📚 Additional Resources

### Books
- "Computer Networking: A Top-Down Approach" - Kurose & Ross
- "TCP/IP Illustrated" - W. Richard Stevens

### Online Resources
- [Kubernetes Networking Documentation](https://kubernetes.io/docs/concepts/services-networking/)
- [AWS VPC Documentation](https://docs.aws.amazon.com/vpc/)
- [Mozilla MDN - HTTP Documentation](https://developer.mozilla.org/en-US/docs/Web/HTTP)

### Tools for Hands-On Practice
- [Katacoda Scenarios](https://katacoda.com/) - Interactive learning environments
- [Docker Playground](https://www.docker.com/play) - No-install Docker environment
- [Kubernetes Playground](https://www.katacoda.com/courses/kubernetes/playground) - Try Kubernetes online

---

## 📝 License

This educational material is provided as-is for learning purposes.

---

## ❓ FAQ

**Q: How long does it take to complete the course?**  
A: Approximately 40-60 hours for comprehensive completion, depending on your background and pace.

**Q: Do I need Kubernetes/AWS experience to start?**  
A: No! Start from Module 00. We cover everything from basics.

**Q: Can I skip modules?**  
A: Modules 0-5 are foundational. You can skip ahead if you're familiar with basics, but modules 9-13 require understanding of earlier concepts.

**Q: Are there video tutorials?**  
A: This course is text-based for better learning retention and offline access.

**Q: How do I get hands-on access to AWS/Kubernetes?**  
A: Use free tiers: [AWS Free Tier](https://aws.amazon.com/free/) or [Kubernetes Playground](https://www.katacoda.com/).

**Q: Can I use this in production environments?**  
A: The commands and concepts shown are educational. Always test in non-production first.

---

## 🎉 Start Your Learning Journey

Ready to master networking? Start with:

```bash
cd 00-setup-and-networking-tools
cat README.md
```

Happy learning! 🚀
