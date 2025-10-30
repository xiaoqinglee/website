---
title: Migrate from Neon Azure Native Integration
subtitle: Learn how to transfer projects and transition from Azure-managed to Neon-managed organizations.
enableTableOfContents: true
redirectFrom:
  - /docs/import/transfer-from-azure-native
updatedOn: '2025-10-29T19:08:44.077Z'
---

<Admonition type="important" title="deprecated">
The Neon Azure Native Integration is deprecated and reaches end of life on **January 31, 2026**. After this date, Azure-managed organizations will no longer be available. [Transfer your projects to a Neon-managed organization](/docs/import/migrate-from-azure-native) to continue using Neon.
</Admonition>

This guide describes how to transfer your projects to your Neon-managed organization to continue using Neon.

## Getting started

Before you begin your migration, be aware of the following:

- You may have more than one organization in the Neon Console. [Review your organizations](#identify-your-organizations) to identify which one is Azure-managed and which is Neon-managed.
- Admins from your Azure-managed organization are automatically added to the Neon-managed organization. Members and project collaborators must be re-added manually after migration.
- If you are on a paid Azure plan, your Neon-managed organization is on the Free plan. You must upgrade to a paid plan (Scale recommended for Azure Scale and Business customers) or create a new paid organization to maintain your current features and avoid service limitations.
- Application connection strings remain the same after transfer because the project structure does not change.
- You can [rename an organization](/docs/manage/orgs-manage#rename-an-organization) at any time.

To transfer your projects to a Neon-managed organization:

<Steps>

## Identify your organizations

1. Sign in to the [Neon Console](https://console.neon.tech).
2. Open the organization dropdown to view your available organizations.
3. Determine which organization is Azure-managed and which is Neon-managed.
   - The Azure-managed organization will have the same name as the resource shown in the Azure Portal.
   - In **Organization Settings** → **Delete**, the Azure-managed organization includes a note that says: "This organization is managed by Azure and can be deleted only from the Azure Portal." A Neon-managed organization will not have this note.

> From the Neon Console, you'll be migrating from the Azure-managed organization to the Neon-managed organization.

## Choose your destination organization

If you have multiple admins, coordinate to decide which Neon-managed organization will be your shared destination:

- **Free plan** users can use their existing Neon-managed organization, or upgrade to a paid plan to create an additional organization (you can only have one free organization).
- **Paid plan** users can upgrade their existing Neon-managed organization, or create a new paid organization.

## Upgrade your Neon plan (paid users only)

If you are on a paid Azure plan, you can either upgrade your existing Neon-managed organization or create a new paid organization:

**To upgrade your existing Neon-managed organization:**

1. Switch to your Neon-managed organization.
2. Go to **Billing** and select **Change plan**.
3. Select **Scale** (recommended for Azure Scale and Business customers) or **Launch**.
4. Complete the upgrade process.

**To create a new paid organization:**

1. From the organization menu, select **Create new organization**.
2. Select **Scale** or **Launch** as your plan.
3. Complete the organization setup.

This ensures your projects retain all paid features after transfer.

## Transfer your projects

You can transfer all projects at once or individually. From your Azure-managed organization in the Neon Console:

1. Go to **Organization** → **Settings** → **Transfer projects**.
2. Click **Select all**, then click **Next**.
3. Choose your Neon-managed organization as the destination.
4. Confirm the transfer.

Projects appear in your destination organization immediately after the transfer completes. For more details about project transfers, see [Transfer projects](/docs/manage/orgs-project-transfer).

## Update your organization configuration

After the transfer:

- Re-add any additional admins, members, or project collaborators who need access. See [Manage organization members](/docs/manage/orgs-manage#add-a-user-to-an-organization) for instructions.
- Verify that all projects appear in your Neon-managed organization.
- For [API keys](/docs/manage/api-keys), project-scoped and personal API keys remain the same after transfer. Organization API keys are tied to your Azure-managed organization, so if you use organization API keys, create new keys in your Neon-managed organization and update them in your applications, scripts, and integrations.
- Update any integrations or tooling that rely on organization-level identifiers.

## Delete your Azure-managed resource

<Admonition type="important">
Only delete your Azure resource after confirming all projects have been transferred. Deleting the Azure resource before transferring projects will permanently delete all projects and data in your Azure-managed organization.
</Admonition>

1. Sign in to the [Azure Portal](https://portal.azure.com).
2. Select your Neon resource created through the Azure Marketplace.
3. Confirm that no projects remain in your Azure-managed organization.
4. On the **Overview** page, select **Delete**.
5. Confirm the deletion by entering the resource's name.
6. Choose the reason for deleting the resource.
7. Select **Delete** to finalize.

Deleting the resource stops all Azure Marketplace billing and completes your transition to a Neon-managed organization.

</Steps>

## After you migrate

Your projects are now managed directly in the Neon Console. All connection strings and project configurations remain the same. You can now manage billing, upgrades, and support directly through Neon.

If you need help, contact Neon Support through the Neon Console or visit the [support documentation](/docs/introduction/support).
