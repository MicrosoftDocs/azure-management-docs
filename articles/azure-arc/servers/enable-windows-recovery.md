---
title: Enable Windows Recovery Environment and Quick Machine Recovery through Azure Arc
description: Learn how to enable Windows Recovery Environment and Quick Machine Recovery through Azure Arc.
ms.date: 10/21/2026
ms.topic: how-to
ai-usage: ai-assisted
---

# Enable Windows Recovery Environment and Quick Machine Recovery through Azure Arc (preview)

Prepare your Arc-enabled Windows servers for startup failures by enabling Windows Recovery Environment (WinRE) on Windows Server 2019 or later. On Windows Server 2025, you can enable Quick Machine Recovery (QMR) in addition to WinRE.

This article explains how to check WinRE status and configure recovery through Azure Arc or Azure Policy. Choose the configuration that meets your recovery needs.

| Configuration | Supported operating systems | What it provides |
|---|---|---|
| **WinRE only** | Windows Server 2019 or later. | Recovery and troubleshooting tools to diagnose and repair common causes of startup failures. |
| **WinRE with QMR** | Windows Server 2025. | WinRE recovery tools, plus the ability to search Windows Update for Microsoft-provided remediations, apply an available fix, and restart the machine after repeated startup failures. |

QMR is a best-effort capability and might not have a remediation for every startup failure.

> [!IMPORTANT]
> This article covers enablement through Azure Arc. For detailed Windows behavior, networking, test mode, recovery partition design, and advanced QMR settings, see the Windows documentation in [Related content](#related-content).

## Prerequisites

- Configure a Windows Recovery Environment (WinRE) partition on your Arc-enabled server. For partition requirements and setup guidance, see [Windows Recovery Environment (Windows RE)](/windows-hardware/manufacture/desktop/windows-recovery-environment--windows-re--technical-reference#hard-drive-partitions).
- To enable WinRE, use Windows Server 2019 or later.
- To enable QMR in addition to WinRE, use Windows Server 2025.

## Check Windows recovery status

Use the Windows Recovery page in Azure Arc to check whether WinRE is enabled on an Arc-enabled server. The status check assigns the audit policy and displays the result in the WinRE status column.

1. In the [Azure portal](https://portal.azure.com/), search for **Azure Arc**.
2. In the table of contents, select **Machine configuration**.
3. Select the **Windows Recovery (preview)** tab.
4. Navigate to the machine whose status you want to check.
5. In the **WinRE status** column for the machine, select **Check status**.
6. In the **Check Windows Recovery Status** pane, select **Audit status** at the bottom.

Alternatively, you can directly apply the [Audit Windows machines that do not have Windows Recovery Environment (WinRE) enabled (preview)](https://ms.portal.azure.com/#view/Microsoft_Azure_Policy/PolicyDetail.ReactView/id/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fea13dcd3-9b62-4fa0-9462-1695a76d0e59/version/1.0.0-preview/scopes~/%5B%22%2Fsubscriptions%2F076b00c5-69cb-4fbf-927c-e3473de84ab5%22%2C%22%2Fsubscriptions%2F84ca48fe-c942-42e5-b492-d56681d058fa%22%2C%22%2Fsubscriptions%2Fa1e43715-e363-4f91-a4d8-b5ea2299cd0c%22%2C%22%2Fsubscriptions%2F6fb4f94a-8da3-4581-b8f1-7e166792b8a2%22%2C%22%2Fsubscriptions%2F879c3e12-a379-45c7-b1b1-5a44f3cc2713%22%2C%22%2Fsubscriptions%2F5423a68c-2f22-4512-a82c-eb6ed44d9aaf%22%2C%22%2Fsubscriptions%2F823ca539-d44d-43ee-8dc8-023fd4f27396%22%2C%22%2Fsubscriptions%2Fb2a328a7-ffff-4c09-b643-a4758cf170bc%22%2C%22%2Fsubscriptions%2Feef8b6d5-94da-4b36-9327-a662f2674efb%22%2C%22%2Fsubscriptions%2F11e4cc61-ad2a-4547-81de-e547594c23fa%22%2C%22%2Fsubscriptions%2Ff7616b56-4c4f-481f-8a80-0e0f51d430c5%22%2C%22%2Fsubscriptions%2F8d5565a3-dec1-4ee2-86d6-8aabb315eec4%22%2C%22%2Fsubscriptions%2Fe0fd569c-e34a-4249-8c24-e8d723c7f054%22%2C%22%2Fsubscriptions%2Ff1d79e73-f8e3-4b10-bfdb-4207ca0723ed%22%2C%22%2Fsubscriptions%2Fe686ef8c-d35d-4e9b-92f8-caaaa7948c0a%22%2C%22%2Fsubscriptions%2F6e967edb-425b-4a33-ae98-f1d2c509dda3%22%2C%22%2Fsubscriptions%2F204898ee-cd13-4332-b9d4-55ca5c25496d%22%2C%22%2Fsubscriptions%2F2145a411-d149-4010-84d4-40fe8a55db44%22%2C%22%2Fsubscriptions%2Fa178425c-491a-416c-b313-39dce68d9b86%22%2C%22%2Fsubscriptions%2Fe2a8a7b3-cb7a-4930-9c79-2b0eb7bf843c%22%2C%22%2Fsubscriptions%2F531c5c86-c959-41ec-91ff-5404a0b217b8%22%2C%22%2Fsubscriptions%2F149b7d43-02c2-4384-822e-592319c408c9%22%2C%22%2Fsubscriptions%2F4c33a6bb-d566-4d0b-8951-00b55a91fba8%22%2C%22%2Fsubscriptions%2F1894bac5-05f1-459a-a19b-e597d701cdb5%22%2C%22%2Fsubscriptions%2F7d07dbdd-8117-46e9-8f58-721da3f32fd4%22%2C%22%2Fsubscriptions%2F326daf21-cbc6-40c8-877f-10324177771e%22%2C%22%2Fsubscriptions%2Fbf86a2b3-d76c-42b8-96a8-03836db5afcb%22%2C%22%2Fsubscriptions%2Fdb7fce4e-7789-4bb0-a1c3-e3b510e8993c%22%2C%22%2Fsubscriptions%2F99b71b51-9458-4a6b-b7a2-9153eb360d42%22%2C%22%2Fsubscrip) policy.

## Enable WinRE with optional QMR

Use the Windows Recovery page in Azure Arc to enable WinRE on supported machines. On Windows Server 2025 or later, you can also enable QMR during the same configuration process. [Check each machine's WinRE status](#check-windows-recovery-status) before you begin.

1. In the [Azure portal](https://portal.azure.com/), search for **Azure Arc**.
1. In the table of contents, select **Machine configuration**.
1. Select the **Windows Recovery (preview)** tab.
1. Select one or more machines that you want to configure.
1. On the command bar, select **Enable Windows Recovery**.
1. In the **Enable Windows Recovery** pane, review the selected machines and their eligibility.

   > [!NOTE]
   > QMR configuration through the Azure Arc experience turns on cloud remediation and automatic remediation for eligible machines. For more information, see [Quick Machine Recovery](/windows/configuration/quick-machine-recovery/).

1. To enable QMR on eligible machines, select **Enable QMR for eligible machines**. Leave the checkbox cleared to enable WinRE only. Machines that aren't eligible for QMR are still configured for WinRE.
1. Select **Enable** at the bottom.

After you enable Windows recovery through Azure Arc, you can identify machines that enter WinRE in the Azure portal. While a machine is in WinRE, its Azure Arc agent status changes to **Recovery**, providing visibility even when Windows can't start normally.

### Configure recovery with Azure Policy

You can also configure recovery across your environment by assigning the following Azure Policy definitions:

- [Configure Windows Recovery Environment (WinRE) on Windows machines (preview)](https://ms.portal.azure.com/#view/Microsoft_Azure_Policy/PolicyDetail.ReactView/id/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fdd970030-1eb5-4d03-b72e-e2c59dcbb202/version/1.0.0-preview/scopes~/%5B%22%2Fsubscriptions%2F076b00c5-69cb-4fbf-927c-e3473de84ab5%22%2C%22%2Fsubscriptions%2F84ca48fe-c942-42e5-b492-d56681d058fa%22%2C%22%2Fsubscriptions%2Fa1e43715-e363-4f91-a4d8-b5ea2299cd0c%22%2C%22%2Fsubscriptions%2F6fb4f94a-8da3-4581-b8f1-7e166792b8a2%22%2C%22%2Fsubscriptions%2F879c3e12-a379-45c7-b1b1-5a44f3cc2713%22%2C%22%2Fsubscriptions%2F5423a68c-2f22-4512-a82c-eb6ed44d9aaf%22%2C%22%2Fsubscriptions%2F823ca539-d44d-43ee-8dc8-023fd4f27396%22%2C%22%2Fsubscriptions%2Fb2a328a7-ffff-4c09-b643-a4758cf170bc%22%2C%22%2Fsubscriptions%2Feef8b6d5-94da-4b36-9327-a662f2674efb%22%2C%22%2Fsubscriptions%2F11e4cc61-ad2a-4547-81de-e547594c23fa%22%2C%22%2Fsubscriptions%2Ff7616b56-4c4f-481f-8a80-0e0f51d430c5%22%2C%22%2Fsubscriptions%2F8d5565a3-dec1-4ee2-86d6-8aabb315eec4%22%2C%22%2Fsubscriptions%2Fe0fd569c-e34a-4249-8c24-e8d723c7f054%22%2C%22%2Fsubscriptions%2Ff1d79e73-f8e3-4b10-bfdb-4207ca0723ed%22%2C%22%2Fsubscriptions%2Fe686ef8c-d35d-4e9b-92f8-caaaa7948c0a%22%2C%22%2Fsubscriptions%2F6e967edb-425b-4a33-ae98-f1d2c509dda3%22%2C%22%2Fsubscriptions%2F204898ee-cd13-4332-b9d4-55ca5c25496d%22%2C%22%2Fsubscriptions%2F2145a411-d149-4010-84d4-40fe8a55db44%22%2C%22%2Fsubscriptions%2Fa178425c-491a-416c-b313-39dce68d9b86%22%2C%22%2Fsubscriptions%2Fe2a8a7b3-cb7a-4930-9c79-2b0eb7bf843c%22%2C%22%2Fsubscriptions%2F531c5c86-c959-41ec-91ff-5404a0b217b8%22%2C%22%2Fsubscriptions%2F149b7d43-02c2-4384-822e-592319c408c9%22%2C%22%2Fsubscriptions%2F4c33a6bb-d566-4d0b-8951-00b55a91fba8%22%2C%22%2Fsubscriptions%2F1894bac5-05f1-459a-a19b-e597d701cdb5%22%2C%22%2Fsubscriptions%2F7d07dbdd-8117-46e9-8f58-721da3f32fd4%22%2C%22%2Fsubscriptions%2F326daf21-cbc6-40c8-877f-10324177771e%22%2C%22%2Fsubscriptions%2Fbf86a2b3-d76c-42b8-96a8-03836db5afcb%22%2C%22%2Fsubscriptions%2Fdb7fce4e-7789-4bb0-a1c3-e3b510e8993c%22%2C%22%2Fsubscriptions%2F99b71b51-9458-4a6b-b7a2-9153eb360d42%22%2C%22%2Fsubscrip)
- [Configure Quick Machine Recovery (QMR) on Windows machines (preview)](https://ms.portal.azure.com/#view/Microsoft_Azure_Policy/PolicyDetail.ReactView/id/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe117c61a-cbc6-4f07-9c63-e9f93e5c0b69/version/1.0.0-preview/scopes~/%5B%22%2Fsubscriptions%2F076b00c5-69cb-4fbf-927c-e3473de84ab5%22%2C%22%2Fsubscriptions%2F84ca48fe-c942-42e5-b492-d56681d058fa%22%2C%22%2Fsubscriptions%2Fa1e43715-e363-4f91-a4d8-b5ea2299cd0c%22%2C%22%2Fsubscriptions%2F6fb4f94a-8da3-4581-b8f1-7e166792b8a2%22%2C%22%2Fsubscriptions%2F879c3e12-a379-45c7-b1b1-5a44f3cc2713%22%2C%22%2Fsubscriptions%2F5423a68c-2f22-4512-a82c-eb6ed44d9aaf%22%2C%22%2Fsubscriptions%2F823ca539-d44d-43ee-8dc8-023fd4f27396%22%2C%22%2Fsubscriptions%2Fb2a328a7-ffff-4c09-b643-a4758cf170bc%22%2C%22%2Fsubscriptions%2Feef8b6d5-94da-4b36-9327-a662f2674efb%22%2C%22%2Fsubscriptions%2F11e4cc61-ad2a-4547-81de-e547594c23fa%22%2C%22%2Fsubscriptions%2Ff7616b56-4c4f-481f-8a80-0e0f51d430c5%22%2C%22%2Fsubscriptions%2F8d5565a3-dec1-4ee2-86d6-8aabb315eec4%22%2C%22%2Fsubscriptions%2Fe0fd569c-e34a-4249-8c24-e8d723c7f054%22%2C%22%2Fsubscriptions%2Ff1d79e73-f8e3-4b10-bfdb-4207ca0723ed%22%2C%22%2Fsubscriptions%2Fe686ef8c-d35d-4e9b-92f8-caaaa7948c0a%22%2C%22%2Fsubscriptions%2F6e967edb-425b-4a33-ae98-f1d2c509dda3%22%2C%22%2Fsubscriptions%2F204898ee-cd13-4332-b9d4-55ca5c25496d%22%2C%22%2Fsubscriptions%2F2145a411-d149-4010-84d4-40fe8a55db44%22%2C%22%2Fsubscriptions%2Fa178425c-491a-416c-b313-39dce68d9b86%22%2C%22%2Fsubscriptions%2Fe2a8a7b3-cb7a-4930-9c79-2b0eb7bf843c%22%2C%22%2Fsubscriptions%2F531c5c86-c959-41ec-91ff-5404a0b217b8%22%2C%22%2Fsubscriptions%2F149b7d43-02c2-4384-822e-592319c408c9%22%2C%22%2Fsubscriptions%2F4c33a6bb-d566-4d0b-8951-00b55a91fba8%22%2C%22%2Fsubscriptions%2F1894bac5-05f1-459a-a19b-e597d701cdb5%22%2C%22%2Fsubscriptions%2F7d07dbdd-8117-46e9-8f58-721da3f32fd4%22%2C%22%2Fsubscriptions%2F326daf21-cbc6-40c8-877f-10324177771e%22%2C%22%2Fsubscriptions%2Fbf86a2b3-d76c-42b8-96a8-03836db5afcb%22%2C%22%2Fsubscriptions%2Fdb7fce4e-7789-4bb0-a1c3-e3b510e8993c%22%2C%22%2Fsubscriptions%2F99b71b51-9458-4a6b-b7a2-9153eb360d42%22%2C%22%2Fsubscrip)

> [!NOTE]
> To enable WinRE only, assign the WinRE policy. On supported machines running Windows Server 2025, the QMR policy enables both WinRE and QMR, so you don't need to assign the WinRE policy separately.

If WinRE is already enabled and you want to add QMR, assign the QMR policy and create a remediation task for the existing machines. You don't need to disable or re-enable WinRE first.

1. Open the policy definition and select **Assign**.
1. Select the subscription or resource group that contains the target Arc-enabled servers.
1. Review the policy parameters and ensure that Arc-enabled machines are included.
1. Configure the assignment's managed identity and grant the permissions required by the policy.
1. Create a remediation task to apply the configuration to existing machines, then create the assignment.
1. Monitor the assignment's compliance results and remediation task status.

For detailed instructions, see [Create a policy assignment](/azure/governance/policy/assign-policy-portal) and [Remediate noncompliant resources](/azure/governance/policy/how-to/remediate-resources).

## Related content

- [Windows Recovery Environment (Windows RE)](/windows-hardware/manufacture/desktop/windows-recovery-environment--windows-re--technical-reference)
- [Quick Machine Recovery](/windows/configuration/quick-machine-recovery/)
