# Recovery Companion - AWS Architecture Diagram

## Visual Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                              RECOVERY COMPANION AWS ARCHITECTURE                 │
└─────────────────────────────────────────────────────────────────────────────────┘

┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│     Patients    │    │   Hospital EHR  │    │  Pharmacy APIs  │
│    (Mobile/Web) │    │     Systems     │    │                 │
└─────────┬───────┘    └─────────┬───────┘    └─────────┬───────┘
          │                      │                      │
          │                      │                      │
          ▼                      │                      │
┌─────────────────┐              │                      │
│   Route 53 DNS  │              │                      │
└─────────┬───────┘              │                      │
          │                      │                      │
          ▼                      │                      │
┌─────────────────┐              │                      │
│   CloudFront    │              │                      │
│      CDN        │              │                      │
└─────────┬───────┘              │                      │
          │                      │                      │
          ▼                      │                      │
┌─────────────────┐              │                      │
│      AWS WAF    │              │                      │
│   (Security)    │              │                      │
└─────────┬───────┘              │                      │
          │                      │                      │
          ▼                      │                      │
┌─────────────────┐              │                      │
│       ALB       │              │                      │
│ Load Balancer   │              │                      │
└─────────┬───────┘              │                      │
          │                      │                      │
          ▼                      │                      │
┌─────────────────┐              │                      │
│  API Gateway    │              │                      │
│   (REST APIs)   │              │                      │
└─────────┬───────┘              │                      │
          │                      │                      │
          ▼                      │                      │
┌─────────────────┐              │                      │
│     Cognito     │              │                      │
│ Authentication  │              │                      │
└─────────┬───────┘              │                      │
          │                      │                      │
          ▼                      │                      │
┌─────────────────────────────────────────────────────────────────────────────────┐
│                           MICROSERVICES LAYER (ECS/Fargate)                     │
├─────────────────┬─────────────────┬─────────────────┬─────────────────┬─────────┤
│   Document      │    Symptom      │   Medication    │    Recovery     │Provider │
│  Processing     │   Assessment    │   Management    │    Tracking     │ Lookup  │
│   Service       │    Service      │    Service      │    Service      │ Service │
└─────────┬───────┴─────────┬───────┴─────────┬───────┴─────────┬───────┴─────────┘
          │                 │                 │                 │
          ▼                 ▼                 ▼                 ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│                              AI/ML SERVICES LAYER                               │
├─────────────────┬─────────────────┬─────────────────┬─────────────────────────┤
│    Textract     │   Comprehend    │     Bedrock     │       Lambda            │
│   (OCR/PDF)     │   Medical NLP   │   (LLM/AI)      │   (AI Processing)       │
└─────────┬───────┴─────────┬───────┴─────────┬───────┴─────────────────────────┘
          │                 │                 │
          ▼                 ▼                 ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│                              DATA STORAGE LAYER                                 │
├─────────────────┬─────────────────┬─────────────────┬─────────────────────────┤
│   PostgreSQL    │  ElastiCache    │       S3        │      AWS Backup         │
│   (RDS Multi-AZ)│   (Redis)       │  (Documents)    │   (Automated Backup)    │
│   Encrypted     │   Session Cache │   Encrypted     │                         │
└─────────┬───────┴─────────┬───────┴─────────┬───────┴─────────────────────────┘
          │                 │                 │
          ▼                 ▼                 ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│                           SECURITY & COMPLIANCE LAYER                           │
├─────────────────┬─────────────────┬─────────────────┬─────────────────────────┤
│      KMS        │     Secrets     │       IAM       │      CloudTrail         │
│ (Encryption)    │    Manager      │   (Access)      │   (Audit Logging)       │
└─────────────────┴─────────────────┴─────────────────┴─────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────────┐
│                           MONITORING & OPERATIONS                                │
├─────────────────┬─────────────────┬─────────────────┬─────────────────────────┤
│   CloudWatch    │     X-Ray       │     Config      │    Personal Health      │
│ (Metrics/Logs)  │   (Tracing)     │  (Compliance)   │     Dashboard           │
└─────────────────┴─────────────────┴─────────────────┴─────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────────┐
│                              NETWORKING (VPC)                                   │
├─────────────────┬─────────────────┬─────────────────┬─────────────────────────┤
│  Public Subnets │ Private Subnets │   NAT Gateway   │    Internet Gateway     │
│   (ALB, NAT)    │ (ECS, RDS)      │                 │                         │
└─────────────────┴─────────────────┴─────────────────┴─────────────────────────┘
```

## Component Details

### **Frontend & CDN**
- **Route 53**: DNS routing with health checks
- **CloudFront**: Global CDN with edge locations
- **S3 Static Website**: React PWA hosting
- **AWS WAF**: DDoS protection and security rules

### **API & Load Balancing**
- **Application Load Balancer**: Layer 7 load balancing
- **API Gateway**: RESTful API management, rate limiting, caching
- **Cognito**: User pools for patient authentication

### **Microservices (ECS/Fargate)**
1. **Document Processing Service**
   - OCR processing for medical documents
   - Medical text extraction and parsing
   - Integration with Textract and Comprehend

2. **Symptom Assessment Service**
   - AI-powered symptom triage
   - Integration with Bedrock for analysis
   - Emergency alert routing

3. **Medication Management Service**
   - Prescription tracking and alerts
   - Pharmacy API integration
   - Dosage calculations

4. **Recovery Tracking Service**
   - Progress monitoring and milestones
   - Daily goal generation
   - Warning pattern detection

5. **Provider Lookup Service**
   - Healthcare provider discovery
   - Distance calculations
   - Emergency facility routing

### **AI/ML Services**
- **Amazon Textract**: OCR for medical documents (PDFs, images)
- **Amazon Comprehend Medical**: Medical entity extraction and analysis
- **Amazon Bedrock**: Large Language Model for symptom analysis and translation
- **AWS Lambda**: Serverless functions for AI processing workflows

### **Data Storage**
- **Amazon RDS (PostgreSQL)**: 
  - Multi-AZ deployment for high availability
  - Encrypted at rest and in transit
  - Automated backups and point-in-time recovery
  
- **Amazon ElastiCache (Redis)**:
  - Session management
  - Frequently accessed data caching
  - Real-time notifications

- **Amazon S3**:
  - Medical document storage
  - Lifecycle policies for cost optimization
  - Cross-region replication for disaster recovery

### **Security & Compliance**
- **AWS KMS**: Encryption key management for HIPAA compliance
- **AWS Secrets Manager**: Secure storage of API keys and credentials
- **AWS IAM**: Role-based access control with least privilege
- **AWS CloudTrail**: Complete audit logging for compliance

### **Monitoring & Operations**
- **Amazon CloudWatch**: Metrics, logs, and custom dashboards
- **AWS X-Ray**: Distributed tracing for performance optimization
- **AWS Config**: Configuration compliance monitoring
- **AWS Personal Health Dashboard**: Service health notifications

### **Networking**
- **VPC**: Isolated network environment
- **Public Subnets**: ALB and NAT Gateway
- **Private Subnets**: ECS services and RDS instances
- **Security Groups**: Firewall rules for each service tier

## Data Flow

1. **Patient Access**: Route 53 → CloudFront → WAF → ALB → API Gateway
2. **Authentication**: API Gateway → Cognito → IAM roles
3. **Service Routing**: API Gateway → ECS microservices
4. **AI Processing**: Services → Textract/Comprehend/Bedrock → Lambda
5. **Data Storage**: Services → RDS/Redis/S3 (all encrypted)
6. **External Integration**: Services → Hospital EHR/Pharmacy APIs
7. **Monitoring**: All services → CloudWatch/X-Ray

## High Availability & Disaster Recovery

- **Multi-AZ Deployment**: RDS and ECS across multiple availability zones
- **Auto Scaling**: ECS services scale based on demand
- **Load Balancing**: ALB distributes traffic across healthy instances
- **Backup Strategy**: Automated daily backups with 30-day retention
- **Disaster Recovery**: Cross-region replication for critical data

## Security Features

- **Encryption**: All data encrypted at rest (KMS) and in transit (TLS 1.3)
- **Network Security**: VPC with private subnets and security groups
- **Access Control**: IAM roles with least privilege access
- **Audit Logging**: CloudTrail logs all API calls and data access
- **Compliance**: HIPAA-compliant infrastructure and processes

## Cost Optimization

- **Reserved Instances**: For predictable workloads (RDS, ECS)
- **Spot Instances**: For non-critical batch processing
- **S3 Lifecycle Policies**: Automatic data archiving to reduce costs
- **CloudWatch**: Monitor and optimize resource utilization
- **Auto Scaling**: Scale resources based on actual demand