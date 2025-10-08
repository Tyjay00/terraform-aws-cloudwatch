# Terraform AWS CloudWatch Module 📊

> **Comprehensive Terraform module for AWS CloudWatch monitoring, alerting, dashboards, and log management**

[![Terraform](https://img.shields.io/badge/Terraform-%E2%89%A5%201.3-623CE4?logo=terraform)](https://terraform.io)
[![AWS Provider](https://img.shields.io/badge/AWS%20Provider-%E2%89%A5%205.0-FF9900?logo=amazon-aws)](https://registry.terraform.io/providers/hashicorp/aws/latest)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

## 🎯 **Overview**

This Terraform module creates a complete AWS CloudWatch monitoring infrastructure with custom metrics, alarms, dashboards, log groups, and automated incident response. Perfect for production monitoring, SLA compliance, and operational excellence.

## 🚀 **Features**

### **Core Monitoring**
- 📊 **Custom Dashboards** - Real-time metric visualization
- 🚨 **Smart Alarms** - Multi-dimensional alerting
- 📝 **Log Management** - Centralized log aggregation
- 📈 **Custom Metrics** - Application-specific monitoring
- 🔔 **SNS Integration** - Multi-channel notifications
- 🎯 **Composite Alarms** - Complex alert conditions

### **Advanced Features**
- 🔍 **Log Insights** - Query and analyze logs
- 📊 **Container Insights** - ECS/EKS monitoring
- 🚀 **Lambda Insights** - Serverless performance
- 🎯 **Synthetics** - Synthetic monitoring
- 📈 **Anomaly Detection** - Machine learning alerts
- 🔄 **Auto Remediation** - Lambda-based responses

## 📋 **Usage**

### **Basic Application Monitoring**
```hcl
module "app_monitoring" {
  source = "./terraform-aws-cloudwatch"

  project_name = "my-web-app"
  environment  = "production"
  
  # Notification Configuration
  notification_email = "alerts@company.com"
  
  # Log Groups
  log_groups = [
    {
      name              = "/aws/lambda/api-function"
      retention_in_days = 30
    },
    {
      name              = "/aws/ecs/web-app"
      retention_in_days = 14
    }
  ]
  
  # Basic Alarms
  alarms = [
    {
      name               = "high-error-rate"
      metric_name        = "ErrorRate"
      namespace          = "AWS/ApplicationELB"
      statistic          = "Average"
      period             = 300
      evaluation_periods = 2
      threshold          = 5
      comparison_operator = "GreaterThanThreshold"
      alarm_description  = "Application error rate is too high"
      
      dimensions = {
        LoadBalancer = "app/my-web-app-alb/1234567890"
      }
    }
  ]
}
```

### **Comprehensive Infrastructure Monitoring**
```hcl
module "infrastructure_monitoring" {
  source = "./terraform-aws-cloudwatch"

  project_name = "infrastructure"
  environment  = "production"
  
  # Multi-channel Notifications
  notification_channels = {
    critical = {
      email = "critical@company.com"
      slack_webhook = "https://hooks.slack.com/services/..."
      pagerduty_endpoint = "https://events.pagerduty.com/..."
    }
    warning = {
      email = "ops@company.com"
    }
  }
  
  # Custom Dashboard
  dashboards = [
    {
      name = "Infrastructure-Overview"
      body = jsonencode({
        widgets = [
          {
            type   = "metric"
            width  = 12
            height = 6
            properties = {
              metrics = [
                ["AWS/EC2", "CPUUtilization", "InstanceId", "i-1234567890abcdef0"],
                ["AWS/RDS", "CPUUtilization", "DBInstanceIdentifier", "mydb-instance"],
                ["AWS/ApplicationELB", "RequestCount", "LoadBalancer", "app/my-alb/1234567890"]
              ]
              period = 300
              stat   = "Average"
              region = "us-east-1"
              title  = "System Performance"
            }
          }
        ]
      })
    }
  ]
  
  # Advanced Alarms
  alarms = [
    # EC2 CPU Alarm
    {
      name                = "ec2-high-cpu"
      metric_name         = "CPUUtilization"
      namespace           = "AWS/EC2"
      statistic           = "Average"
      period              = 300
      evaluation_periods  = 3
      threshold           = 80
      comparison_operator = "GreaterThanThreshold"
      alarm_description   = "EC2 instance CPU utilization is high"
      severity            = "warning"
      
      dimensions = {
        InstanceId = "i-1234567890abcdef0"
      }
    },
    
    # RDS Connection Alarm
    {
      name                = "rds-connection-count"
      metric_name         = "DatabaseConnections"
      namespace           = "AWS/RDS"
      statistic           = "Average"
      period              = 300
      evaluation_periods  = 2
      threshold           = 80
      comparison_operator = "GreaterThanThreshold"
      alarm_description   = "RDS connection count is approaching limit"
      severity            = "critical"
      
      dimensions = {
        DBInstanceIdentifier = "mydb-instance"
      }
    },
    
    # Application Load Balancer Response Time
    {
      name                = "alb-response-time"
      metric_name         = "TargetResponseTime"
      namespace           = "AWS/ApplicationELB"
      statistic           = "Average"
      period              = 300
      evaluation_periods  = 2
      threshold           = 2
      comparison_operator = "GreaterThanThreshold"
      alarm_description   = "Application response time is too high"
      severity            = "warning"
      
      dimensions = {
        LoadBalancer = "app/my-alb/1234567890"
      }
    }
  ]
  
  # Log Metric Filters
  log_metric_filters = [
    {
      name           = "error-count"
      log_group_name = "/aws/lambda/api-function"
      filter_pattern = "ERROR"
      metric_transformation = {
        name      = "ErrorCount"
        namespace = "MyApp/Lambda"
        value     = "1"
      }
    }
  ]
}
```

### **Microservices Monitoring with Service Map**
```hcl
module "microservices_monitoring" {
  source = "./terraform-aws-cloudwatch"

  project_name = "microservices"
  environment  = "production"
  
  # Service-specific Log Groups
  log_groups = [
    {
      name              = "/ecs/user-service"
      retention_in_days = 30
      kms_key_id        = aws_kms_key.logs.arn
    },
    {
      name              = "/ecs/order-service"
      retention_in_days = 30
      kms_key_id        = aws_kms_key.logs.arn
    },
    {
      name              = "/ecs/payment-service"
      retention_in_days = 90  # Longer retention for financial data
      kms_key_id        = aws_kms_key.logs.arn
    }
  ]
  
  # Composite Alarms for Service Health
  composite_alarms = [
    {
      name                = "user-service-health"
      alarm_description   = "Overall health of user service"
      actions_enabled     = true
      alarm_rule          = join(" OR ", [
        "ALARM(user-service-high-error-rate)",
        "ALARM(user-service-high-latency)",
        "ALARM(user-service-low-healthy-hosts)"
      ])
    }
  ]
  
  # Custom Metrics for Business KPIs
  custom_metrics = [
    {
      namespace    = "MyApp/Business"
      metric_name  = "SignupRate"
      unit         = "Count/Second"
      description  = "User signup rate"
    },
    {
      namespace    = "MyApp/Business"
      metric_name  = "RevenuePerMinute"
      unit         = "None"
      description  = "Revenue generated per minute"
    }
  ]
  
  # Anomaly Detection
  anomaly_detectors = [
    {
      metric_name = "RequestCount"
      namespace   = "AWS/ApplicationELB"
      stat        = "Average"
      dimensions = {
        LoadBalancer = "app/api-gateway-alb/1234567890"
      }
    }
  ]
}
```

### **Cost Monitoring and Optimization**
```hcl
module "cost_monitoring" {
  source = "./terraform-aws-cloudwatch"

  project_name = "cost-management"
  environment  = "production"
  
  # Billing Alarms
  billing_alarms = [
    {
      name                = "monthly-spend-warning"
      threshold           = 1000
      comparison_operator = "GreaterThanThreshold"
      evaluation_periods  = 1
      period              = 86400  # Daily
      alarm_description   = "Monthly AWS spend approaching budget"
    },
    {
      name                = "monthly-spend-critical"
      threshold           = 1500
      comparison_operator = "GreaterThanThreshold"
      evaluation_periods  = 1
      period              = 86400
      alarm_description   = "Monthly AWS spend exceeded budget"
      severity            = "critical"
    }
  ]
  
  # Resource Utilization Monitoring
  resource_optimization_alarms = [
    {
      name                = "underutilized-ec2"
      metric_name         = "CPUUtilization"
      namespace           = "AWS/EC2"
      statistic           = "Average"
      period              = 3600   # 1 hour
      evaluation_periods  = 24     # 24 hours
      threshold           = 10     # Less than 10% CPU
      comparison_operator = "LessThanThreshold"
      alarm_description   = "EC2 instance is underutilized"
      treat_missing_data  = "notBreaching"
    }
  ]
}
```

## 📝 **Input Variables**

### **Required Variables**
| Name | Description | Type |
|------|-------------|------|
| `project_name` | Name of the project | `string` |
| `environment` | Environment (prod/staging/dev) | `string` |

### **Notification Configuration**
| Name | Description | Type | Default |
|------|-------------|------|---------|
| `notification_email` | Email for alarm notifications | `string` | `""` |
| `notification_channels` | Multi-channel notification config | `map(object)` | `{}` |
| `sns_topic_name` | Custom SNS topic name | `string` | `""` |

### **Log Management**
| Name | Description | Type | Default |
|------|-------------|------|---------|
| `log_groups` | List of CloudWatch log groups | `list(object)` | `[]` |
| `log_retention_days` | Default log retention period | `number` | `14` |
| `enable_log_insights` | Enable CloudWatch Logs Insights | `bool` | `true` |

### **Alarms Configuration**
| Name | Description | Type | Default |
|------|-------------|------|---------|
| `alarms` | List of CloudWatch alarms | `list(object)` | `[]` |
| `composite_alarms` | List of composite alarms | `list(object)` | `[]` |
| `enable_anomaly_detection` | Enable anomaly detection | `bool` | `false` |

### **Dashboard Configuration**
| Name | Description | Type | Default |
|------|-------------|------|---------|
| `dashboards` | List of CloudWatch dashboards | `list(object)` | `[]` |
| `create_default_dashboard` | Create default dashboard | `bool` | `true` |

## 📤 **Outputs**

| Name | Description |
|------|-------------|
| `sns_topic_arn` | SNS topic ARN for notifications |
| `log_group_names` | Names of created log groups |
| `log_group_arns` | ARNs of created log groups |
| `alarm_names` | Names of created alarms |
| `alarm_arns` | ARNs of created alarms |
| `dashboard_urls` | URLs of created dashboards |
| `metric_filter_names` | Names of created metric filters |

## 🏗️ **Architecture**

```
                    ┌─────────────────┐
                    │   Applications  │
                    │   & Services    │
                    └─────────┬───────┘
                              │
                              ▼
                    ┌─────────────────┐
                    │   CloudWatch    │
                    │     Logs        │
                    └─────────┬───────┘
                              │
        ┌─────────────────────┼─────────────────────┐
        │                     │                     │
        ▼                     ▼                     ▼
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Metric    │     │   Alarms    │     │ Dashboards  │
│  Filters    │     │ & Anomaly   │     │ & Insights  │
│             │     │ Detection   │     │             │
└─────────┬───┘     └─────────┬───┘     └─────────────┘
          │                   │
          ▼                   ▼
┌─────────────┐     ┌─────────────┐
│   Custom    │     │     SNS     │
│   Metrics   │     │ Notifications│
└─────────────┘     └─────────┬───┘
                              │
        ┌─────────────────────┼─────────────────────┐
        │                     │                     │
        ▼                     ▼                     ▼
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│    Email    │     │    Slack    │     │ PagerDuty   │
│ Notifications│     │   Webhooks  │     │  Incidents  │
└─────────────┘     └─────────────┘     └─────────────┘
```

## 🔒 **Security & Compliance**

### **Data Protection**
- 🔐 **Encryption at Rest** - KMS encryption for logs
- 🔒 **Encryption in Transit** - TLS for all communications
- 🛡️ **Access Control** - IAM policies for resources
- 📊 **Audit Logging** - CloudTrail integration

### **Compliance Features**
- 📋 **Retention Policies** - Configurable log retention
- 🔍 **Access Monitoring** - Who accessed what data
- 📈 **Compliance Dashboards** - Regulatory metrics
- 🚨 **Breach Detection** - Unusual access patterns

## 💰 **Cost Optimization**

### **Pricing Components**
- **Logs Ingestion**: $0.50 per GB ingested
- **Logs Storage**: $0.03 per GB per month
- **Custom Metrics**: $0.30 per metric per month
- **API Requests**: $0.01 per 1,000 requests
- **Dashboards**: $3.00 per dashboard per month

### **Cost-Saving Strategies**
- 📊 **Log Retention** - Optimize retention periods
- 🎯 **Metric Filtering** - Reduce unnecessary metrics
- 📈 **Sampling** - Sample high-volume logs
- 🔄 **Log Compression** - Compress before ingestion

## 🧪 **Examples**

Check the [examples](examples/) directory for complete implementations:

- **[Web Application](examples/web-app-monitoring/)** - Complete web app monitoring
- **[Microservices](examples/microservices-monitoring/)** - Service mesh monitoring
- **[Database Monitoring](examples/database-monitoring/)** - RDS and DynamoDB monitoring
- **[Cost Optimization](examples/cost-monitoring/)** - Billing and resource monitoring

## 🔧 **Requirements**

| Name | Version |
|------|---------|
| terraform | >= 1.3.0 |
| aws | >= 5.0 |

## 🧪 **Testing**

```bash
# Validate Terraform configuration
terraform validate

# Test alarm functionality
aws cloudwatch set-alarm-state \
  --alarm-name "test-alarm" \
  --state-value ALARM \
  --state-reason "Testing alarm"

# Query logs
aws logs start-query \
  --log-group-name "/aws/lambda/my-function" \
  --start-time 1609459200 \
  --end-time 1609462800 \
  --query-string 'fields @timestamp, @message | filter @message like /ERROR/'
```

## 📊 **Best Practices**

### **Monitoring Strategy**
- 🎯 **Start with SLIs** - Service Level Indicators first
- 📈 **Use Error Budgets** - Define acceptable failure rates
- 🔄 **Monitor the Monitors** - Alert on monitoring failures
- 📊 **Dashboard Hierarchy** - Executive, operational, diagnostic views

### **Alerting Guidelines**
- 🚨 **Alert on Symptoms** - Not just causes
- 📧 **Notification Fatigue** - Reduce false positives
- 🎯 **Actionable Alerts** - Include remediation steps
- 🔄 **Alert Lifecycle** - Acknowledge, investigate, resolve

### **Performance Optimization**
- 📊 **Batch Metrics** - Reduce API calls
- 🎯 **Efficient Queries** - Optimize Log Insights queries
- 📈 **Metric Math** - Use CloudWatch math for calculations
- 🔄 **Resource Tagging** - Organize monitoring resources

## 🤝 **Contributing**

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/monitoring-enhancement`)
3. Commit your changes (`git commit -m 'Add monitoring enhancement'`)
4. Push to the branch (`git push origin feature/monitoring-enhancement`)
5. Open a Pull Request

## 📄 **License**

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🏆 **Related Modules**

- **[terraform-aws-s3](../terraform-aws-s3)** - S3 access logging
- **[terraform-aws-lambda](../terraform-aws-lambda)** - Function monitoring
- **[terraform-aws-ecs](../terraform-aws-ecs)** - Container monitoring
- **[terraform-aws-networking](../terraform-aws-networking)** - Network monitoring

---

**📊 Built for enterprise observability and operational excellence**

> *This module demonstrates advanced CloudWatch architecture patterns and monitoring expertise suitable for production environments with comprehensive SLA management.*