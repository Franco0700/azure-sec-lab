# 🛡️ Secure Cloud Infrastructure & Automated Threat Detection Lab

## 📖 Project Description
This project demonstrates the complete DevSecOps lifecycle and Cloud Security Operations (SecOps). It features the automated deployment of secure infrastructure in Microsoft Azure using **Terraform**, continuous security validations through **GitHub Actions**, and active threat monitoring with **Microsoft Sentinel** (SIEM).

## 🚀 Technologies Used
- **Cloud Provider:** Microsoft Azure
- **Infrastructure as Code (IaC):** Terraform
- **CI/CD & Automation:** GitHub Actions, Linux (Ubuntu), Bash
- **Static Security Analysis:** tfsec
- **SIEM & Monitoring:** Microsoft Sentinel, Azure Monitor Agent (AMA), KQL (Kusto Query Language)

## 🏗️ Architecture

```mermaid
graph TD
    subgraph CI/CD Pipeline
        GA[GitHub Actions] -->|Runs tfsec & terraform apply| TF[Terraform]
    end

    subgraph Microsoft Azure
        TF -->|Deploys| RG[Resource Group: rg-sec-lab]
        
        subgraph Virtual Network
            NSG[Network Security Group <br/> Inbound TCP 22 restricted to Public IP] 
            VM[Ubuntu Linux VM <br/> Password Auth Disabled]
            NSG --- VM
        end
        
        subgraph SIEM & Monitoring
            AMA[Azure Monitor Agent <br/> Data Collection Rule: auth/authpriv]
            LAW[Log Analytics Workspace]
            SEN[Microsoft Sentinel]
            
            VM -->|Syslog Events| AMA
            AMA -->|Ingests| LAW
            LAW --- SEN
        end
        
        RG -.-> Virtual Network
        RG -.-> SIEM & Monitoring
    end
```

The architecture implements the Principle of Least Privilege by restricting administrative access strictly to a trusted public IP address and ingesting only critical authentication logs (Facility: auth/authpriv) to optimize SIEM costs.

🔬 Use Case: Attack Simulation & Threat Hunting
To validate the monitoring pipeline, I executed a simulated SSH brute-force attack against the Linux server.

1. Attack Generation: Repeated login attempts using invalid credentials.2. Log Ingestion: The Azure Monitor Agent (AMA) processed the local auth.log events and forwarded them to the Log Analytics Workspace via Data Collection Rules (DCR).3. Detection (Threat Hunting with KQL):
Using the following KQL query in Sentinel, I successfully isolated the attack attempts and extracted the attacker's IP address for further incident response:

```
Syslog
| where Facility in ("auth", "authpriv")
| where SyslogMessage has_any ("Failed password", "Invalid user")
| project TimeGenerated, Computer, SyslogMessage, SourceIP = extract("from ([0-9.]+)", 1, SyslogMessage)
```