# SMTP Email Sending API (Proof of Concept)

## Overview

This project is a **Proof of Concept (POC) API** designed to simplify and optimize the process of sending emails through various email service providers. The API acts as a single interface that allows users to send emails using different providers such as Amazon SES, Google SMTP, Mailgun, and others, without needing to integrate each service separately.

The key objective of this POC is to demonstrate how businesses, startups, and developers can leverage a unified API to manage their email-sending needs. The API dynamically selects the best email service provider based on pre-defined factors like cost, reliability, and performance, thus offering a flexible, efficient, and cost-effective solution for managing email delivery.

## What This API Does

- **Unified Email Sending Interface**: The API offers a single, consistent endpoint to send emails, regardless of the email service provider being used.
- **Multiple Provider Support**: Users can configure multiple email providers, and the API will choose the most appropriate one for each email based on predefined criteria.
- **Dynamic Routing**: The API dynamically routes emails to the best service provider, evaluating factors such as:
  - Cost per email
  - Delivery performance (success rates, latency, etc.)
  - Provider availability and rate limits
- **Email Attachments**: The API supports sending emails with or without attachments, handling the storage and retrieval of files securely.
- **Cost and Performance Optimization**: By monitoring the cost and performance of each email service provider, the API helps users control their email-sending expenses while ensuring reliable delivery.
- **API Key Security**: Secure API key management is implemented to ensure only authorized users can send emails via the API.

## Why a Proof of Concept SMTP API is Important

Managing email delivery through a single service provider can have limitations. Providers may experience downtime, rate limits, or cost inefficiencies that can affect the reliability of email delivery. This proof of concept aims to showcase the following benefits:

1. **Cost Efficiency**: By allowing businesses to dynamically choose the most cost-effective provider for each email, this API helps reduce expenses for organizations that rely heavily on email communications, such as marketing campaigns or transactional emails.
  
2. **Reliability**: In the event of a provider experiencing downtime, the API can automatically reroute emails through an alternative provider, ensuring high availability and minimizing the risk of failed deliveries.
  
3. **Flexibility**: Users are not tied to one provider and can easily switch between services or use multiple providers based on their needs. This flexibility allows businesses to optimize email performance and scale as needed.

4. **Scalability**: As businesses grow and their email volume increases, this API scales by distributing email-sending tasks across different providers, preventing bottlenecks or delays.

5. **Ease of Use**: The API abstracts away the complexity of integrating with multiple email providers. Developers only need to interact with one API to access various services, streamlining the integration process.

## Use Cases

This API is ideal for:

- **Startups and Small Businesses**: Organizations that need an easy way to send emails without being locked into a single provider, and that want to optimize costs as they grow.
  
- **Marketing Teams**: Teams that send large volumes of promotional emails and need a cost-effective solution to manage email campaigns across multiple regions and providers.

- **Developers and SaaS Platforms**: Developers who want to integrate email-sending functionality into their apps or services without dealing with the complexity of multiple provider APIs.
## Database Schema

The API uses a database to store important information related to email transactions, provider configurations, and performance metrics. Below is an overview of the key entities in the database and their relationships:

### Tables:
```

-- 1. Users
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR(100) UNIQUE NOT NULL,
  password_hash TEXT NOT NULL,
  created_at TIMESTAMP DEFAULT NOW()
);

-- 2. Email Providers
CREATE TABLE providers (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(100) NOT NULL, -- e.g., 'SES', 'Mailgun'
  api_key TEXT NOT NULL, -- Encrypted API Key to use the service
  base_cost DECIMAL(10, 2) NOT NULL, -- Base cost per email
  reliability_score FLOAT DEFAULT 0.0, -- Performance score
  status BOOLEAN DEFAULT TRUE, -- Active or Inactive
  created_at TIMESTAMP DEFAULT NOW()
);

-- 3. Email Templates
CREATE TABLE email_templates (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id),
  template_name VARCHAR(100) NOT NULL,
  template_body TEXT NOT NULL,
  created_at TIMESTAMP DEFAULT NOW()
);

-- 4. Email Jobs
CREATE TABLE email_jobs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id),
  recipient VARCHAR(100) NOT NULL,
  subject VARCHAR(255) NOT NULL,
  body TEXT NOT NULL,
  attachment_url TEXT, -- URL to the attachment if provided
  provider_used UUID REFERENCES providers(id),
  status VARCHAR(50), -- e.g., 'sent', 'failed', 'queued'
  created_at TIMESTAMP DEFAULT NOW(),
  sent_at TIMESTAMP
);

-- 5. Routing Rules
CREATE TABLE routing_rules (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email_type VARCHAR(50) NOT NULL, -- e.g., 'transactional', 'marketing'
  default_provider UUID REFERENCES providers(id),
  fallback_providers UUID[] REFERENCES providers(id), -- Array of fallback providers
  priority INTEGER DEFAULT 1, -- Priority level of the rule
  created_at TIMESTAMP DEFAULT NOW()
);

-- 6. Provider Performance
CREATE TABLE provider_performance (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  provider_id UUID REFERENCES providers(id),
  emails_sent INTEGER DEFAULT 0,
  emails_failed INTEGER DEFAULT 0,
  average_latency FLOAT DEFAULT 0.0,
  last_updated TIMESTAMP DEFAULT NOW()
);

-- 7. API Keys
CREATE TABLE api_keys (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id),
  key TEXT NOT NULL, -- API Key string for authentication
  created_at TIMESTAMP DEFAULT NOW(),
  expires_at TIMESTAMP
);

-- 8. Provider Quota
CREATE TABLE provider_quota (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  provider_id UUID REFERENCES providers(id),
  daily_quota INTEGER,
  monthly_quota INTEGER,
  used_today INTEGER DEFAULT 0,
  used_this_month INTEGER DEFAULT 0,
  reset_at TIMESTAMP -- When the quota resets
);

-- 9. Email Status
CREATE TABLE email_status (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email_job_id UUID REFERENCES email_jobs(id),
  status VARCHAR(50), -- 'delivered', 'bounced', 'failed'
  timestamp TIMESTAMP DEFAULT NOW()
);
```

These tables are essential for managing the email routing, provider selection, and performance tracking aspects of the API. View DB design [here](https://drawsql.app/teams/bahati-corp/diagrams/smtp-api-db-design)

## Conclusion

This POC demonstrates the potential of a unified SMTP email-sending API for organizations of all sizes. By abstracting the complexity of working with different providers, optimizing for cost and performance, and providing a secure, reliable interface for email delivery, this API offers a robust solution for modern email infrastructure needs.
