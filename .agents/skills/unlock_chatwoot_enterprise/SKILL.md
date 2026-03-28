---
name: Unlock Chatwoot Enterprise
description: Automates the process of unlocking Enterprise Edition (EE) features in Chatwoot by modifying key application check points and configuration overrides.
---

# Unlock Chatwoot Enterprise Skill

## Overview
This skill guides the AI to bypass Chatwoot's Community Edition limits and unlock all premium/Enterprise features. It details which files to modify to override the `pricing_plan`, `enterprise?` flags, and frontend limitations.

## Context
Chatwoot's core logic makes several checks to determine if the currently running instance is an Enterprise edition or if it's restricted to Community limitations. To activate all EE functionalities without a valid EE license check, specific Ruby classes overriding limits must be hardcoded.

## Steps

### 1. Bypass `ChatwootApp` Enterprise Check
**File:** `lib/chatwoot_app.rb`

You must ensure that the methods `enterprise?` and `self_hosted_enterprise?` return `true` unconditionally:

```ruby
  def self.enterprise?
    true
  end

  def self.self_hosted_enterprise?
    true
  end
```

*Reasoning: This bypasses environment variable checks (`DISABLE_ENTERPRISE`) and validates the server instance as enterprise for backend logic.*

### 2. Bypass `ChatwootHub` Pricing Plan Validations
**File:** `lib/chatwoot_hub.rb`

You must hardcode the pricing plan string and the license quantity limit:

```ruby
  def self.pricing_plan
    'enterprise'
  end

  def self.pricing_plan_quantity
    999999
  end
```

*Reasoning: Returning `'enterprise'` (or `'premium'`) blocks `ReconcilePlanConfigService` from downgrading features and prevents `User.count >= ChatwootHub.pricing_plan_quantity` verifications in `enterprise/app/models/enterprise/concerns/user.rb` from hindering the creation of new users.*

### 3. Override Dashboard Global Configuration (Frontend)
**File:** `app/controllers/dashboard_controller.rb`

Inject the enterprise pricing plan into the `app_config` payload so the frontend Vue app recognizes the system as premium:

```ruby
  def app_config
    {
      ...
      IS_ENTERPRISE: ChatwootApp.enterprise?,
      INSTALLATION_PRICING_PLAN: 'enterprise',
      ...
    }
  end
```

*Reasoning: The frontend Vue components check `@global_config['INSTALLATION_PRICING_PLAN']` when generating `window.chatwootConfig.enterprisePlanName`. If this falls back to `'community'`, parts of the dashboard UI will remain locked or prompt upgrades.*

## Usage
Whenever the user asks to unlock Chatwoot or remove its enterprise blocks, read this `SKILL.md` and replicate the file modifications precisely as illustrated above. Always remember to check if the `enterprise/` folder exists first, as the Chatwoot structure might heavily depend on its presence to load premium features overriding logic.
