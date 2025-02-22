---
title: Debug copy activity in your SAP CDC solution (preview) by sending logs
titleSuffix: Azure Data Factory
description: Learn how to debug issues with the Azure Data Factory copy activity for your SAP change data capture (CDC) solution (preview) by sending self-hosted integration runtime logs to Microsoft.
author: ukchrist
ms.service: data-factory
ms.subservice: data-movement
ms.topic: conceptual
ms.date: 06/01/2022
ms.author: ulrichchrist
---

# Debug copy activity by sending self-hosted integration runtime logs

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

If you want Microsoft to debug Azure Data Factory copy activity issues in your SAP change data capture (CDC) solution (preview), send us your self-hosted integration runtime logs, and then contact us.

## Send logs to Microsoft

1. On the computer running the self-hosted integration runtime, open Microsoft Integration Runtime Configuration Manager.

1. Select the **Diagnostics** tab. Under **Logging**, select **Send logs**.

   :::image type="content" source="media/sap-change-data-capture-solution/sap-cdc-shir-diagnostics-send-logs.png" alt-text="Screenshot of the Integration Runtime Configuration Manager Diagnostics tab, with Send logs highlighted.":::

1. Enter or select the information that's requested, and then select **Send logs**.

## Contact Microsoft support

After you've uploaded and sent your self-hosted integration runtime logs, contact Microsoft support. In your support request, include the Report ID and Timestamp values that are shown in the confirmation:

:::image type="content" source="media/sap-change-data-capture-solution/sap-cdc-diagnostics-report-id.png" alt-text="Screenshot of the self-hosted integration runtime's diagnostic log confirmation, with Report ID and Timestamp highlighted.":::

## Next steps

[Auto-generate a pipeline by using the SAP data partitioning template](sap-change-data-capture-data-partitioning-template.md)
