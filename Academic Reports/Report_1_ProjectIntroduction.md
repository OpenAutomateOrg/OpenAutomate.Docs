# Project Introduction

## 1. Overview

### 1.1 Project Information
- **Project name:** OpenAutomate - Multi-Tenant Business Process Automation Management Platform
- **Project code:** OABPAMP
- **Group name:** Capstone-Group-5
- **Software type:** Web Application, Desktop Application (Bot Agent)

### 1.2 Project Team

| Full Name | Role | Email | Mobile |
|-----------|------|-------|--------|
| Academic Advisor | Lecturer | advisor@university.edu | - |
| Hoai | Backend Lead / Team Leader | hoai@example.com | - |
| Nhat | Frontend Lead | nhat@example.com | - |
| Chinh | DevOps Specialist | chinh@example.com | - |
| Hung | QA Lead | hung@example.com | - |
| Vu | Bot Agent Lead | vu@example.com | - |

## 2. Product Background

In today's business environment, automation processes have become essential for efficiency and competitiveness. Many organizations are dependent on commercial automation platforms that require substantial licensing costs, specialized training, and create vendor lock-in situations. These platforms often limit flexibility and scalability while requiring significant investment in proprietary technologies.

Organizations face several challenges with existing automation solutions:
- High costs for commercial licenses, sometimes reaching tens of thousands of dollars annually
- Dependency on vendor-specific technologies that restrict innovation
- Limited control over automation assets and processes
- Complex management of distributed automation agents
- Difficulty integrating with existing systems
- Lack of multi-tenant capabilities for organizations with multiple departments or clients

IT departments often struggle to justify the high costs of commercial automation platforms, especially for small to medium businesses, while still needing to deliver the efficiency benefits of automation to their organizations.

## 3. Existing Systems

### 3.1 UiPath

UiPath is a leading commercial Robotic Process Automation (RPA) platform that allows organizations to automate repetitive tasks.

**Features:**
- Visual workflow designer
- Robot management
- Process recording
- Centralized automation hub
- Enterprise-grade security

**Pros:**
- Comprehensive automation capabilities
- Strong enterprise support
- Large community and marketplace
- Robust orchestration

**Cons:**
- High licensing costs
- Requires significant training
- Vendor lock-in
- Limited customization for specific business needs
- Not designed for multi-tenant environments

### 3.2 Automation Anywhere

Automation Anywhere is an enterprise RPA platform offering end-to-end automation for business processes.

**Features:**
- Web-based control center
- Bot insight analytics
- IQ Bot for unstructured data
- Workload management

**Pros:**
- Strong analytics capabilities
- Good scalability
- Cognitive automation features
- Cloud and on-premises deployment options

**Cons:**
- Complex deployment process
- Steep learning curve
- Expensive licensing model
- Limited bot agent customization
- No native multi-tenant architecture

### 3.3 Microsoft Power Automate

Power Automate (formerly Flow) is Microsoft's workflow automation solution integrated with Office 365 and Dynamics.

**Features:**
- Hundreds of connectors for various services
- Desktop and cloud flows
- AI Builder integration
- Process advisor

**Pros:**
- Deep integration with Microsoft ecosystem
- User-friendly interface
- Relatively lower entry cost
- Regular updates and improvements

**Cons:**
- Limited capabilities outside Microsoft ecosystem
- Scaling challenges for complex processes
- Not designed for heavy computational tasks
- Lacks advanced orchestration features
- Limited multi-tenant capabilities

## 4. Business Opportunity

The market for business process automation solutions is growing rapidly, expected to reach $19.6 billion by 2026, with a CAGR of 13.8%. Despite this growth, many organizations are dissatisfied with existing solutions due to high costs, inflexibility, and vendor lock-in.

OpenAutomate addresses these challenges by providing an open-source alternative for business process automation management. By leveraging the accessibility and extensive library ecosystem of Python for automation scripts, organizations can significantly reduce costs while gaining greater control over their automation solutions. This approach eliminates vendor dependency and empowers teams to create, modify, and extend automation processes without the constraints of commercial licensing.

The platform's multi-tenant architecture provides additional value by allowing:
- Managed service providers to host automation solutions for multiple clients
- Large enterprises to segment automation by department or business unit
- Software vendors to embed automation capabilities in their own products

The flexibility of an open-source approach, combined with modern technologies like ASP.NET Core and Next.js, positions OpenAutomate to capture significant market share from organizations looking for:
1. Cost-effective alternatives to expensive commercial platforms
2. Greater control over automation implementations
3. Modern, scalable architecture that supports multi-tenant deployments
4. Open integration capabilities with existing systems

## 5. Software Product Vision

For organizations seeking efficient automation without vendor lock-in, OpenAutomate is an open-source, multi-tenant business process automation management platform that enables teams to create, deploy, monitor, and orchestrate automation processes across the enterprise. Unlike proprietary automation platforms that impose high licensing costs and restrict customization, OpenAutomate provides a flexible, extensible foundation that scales with organizational needs while maintaining complete control over automation assets and implementation.

OpenAutomate empowers teams to build automation solutions using familiar technologies, supports seamless integration with existing systems, and delivers comprehensive monitoring and management capabilities—all without the financial burden of commercial licensing. By democratizing access to enterprise-grade automation, OpenAutomate helps organizations of all sizes achieve operational efficiency, reduce manual effort, and focus human resources on high-value activities.

## 6. Project Scope & Limitations

### 6.1 Major Features

**FE-01: Multi-Tenant Architecture**
- Secure isolation between organization units in a shared infrastructure
- Tenant-specific data access through global query filters
- Tenant resolution through URL path
- Cross-tenant operations for system administrators

**FE-02: Bot Agent Management**
- Deploy and register automation agents across an organization
- Monitor bot agent status and performance in real-time
- Secure communication between server and bot agents via SignalR
- Machine key authentication for bot agents

**FE-03: Automation Package Management**
- Create, version, and deploy automation packages
- Secure storage and distribution of automation files
- Package metadata management and search
- Version tracking and rollback capabilities

**FE-04: Execution Management**
- Schedule automation tasks using cron expressions
- Track execution status and results
- Detailed logging of automation runs
- Error handling and retry mechanisms

**FE-05: Asset Management**
- Define and manage automation assets (databases, applications, credentials)
- Associate assets with bot agents for access control
- Secure storage of sensitive connection information
- Asset grouping and categorization

**FE-06: User and Role Management**
- Secure authentication with JWT and refresh tokens
- Role-based access control for platform features
- Organization unit-specific permissions
- Audit logging of user actions

**FE-07: Monitoring and Analytics**
- Real-time dashboards for automation status
- Historical performance analytics
- Resource utilization monitoring
- Customizable alerts and notifications

### 6.2 Limitations & Exclusions

**LI-1:** The platform focuses on orchestration and management rather than providing a visual automation design tool. Automation scripts will be developed externally using Python.

**LI-2:** Initial version will not support cross-platform bot agents; Windows will be the primary supported operating system for bot agents.

**LI-3:** OpenAutomate is not designed for unattended web automation that requires complex browser interaction; it is focused on back-office process automation.

**LI-4:** The platform will not include native AI/ML capabilities for automation; these would need to be implemented within individual automation scripts.

**LI-5:** Initial release will not include a marketplace for sharing automation packages across organizations.

**LI-6:** The system will not provide built-in OCR or document processing capabilities; these would need to be implemented through external libraries in automation scripts. 