---
title: Capture a browser trace for troubleshooting
description: Capture network information from a browser trace to help troubleshoot issues with the Azure portal.
ms.date: 09/08/2025
ms.topic: how-to
# Customer intent: "As an Azure portal user, I want to capture a browser trace for troubleshooting issues in the Azure portal, so that I can provide detailed information to Microsoft support to resolve the problem effectively."
---

# Capture a browser trace for troubleshooting

If you're troubleshooting an issue with the Azure portal, and you need to contact Microsoft support, you may want to first capture some additional information. For example, it can be helpful to share a browser trace, a step recording, and console output. This information can provide important details about what exactly is happening in the portal when your issue occurs.

> [!WARNING]
> Browser traces often contain sensitive information and might include authentication tokens linked to your identity. We generally recommend ensuring that sensitive information is not included in any trace files that you share.
>
> In certain cases, such as when investigating issues related to signing in to Azure, Microsoft support may request a trace file that includes this sensitive information. Microsoft support uses these traces for troubleshooting purposes only.

You can capture a browser trace in any [supported browser](azure-portal-supported-browsers-devices.md#recommended-browsers): Microsoft Edge, Google Chrome, Safari (on Mac), or Firefox. Steps for each browser are shown below.

## Microsoft Edge

The following steps show how to use the developer tools in Microsoft Edge to capture a browser trace. For more information, see [Microsoft Edge DevTools](/microsoft-edge/devtools-guide-chromium/overview).

1. Sign in to the [Azure portal](https://portal.azure.com). It's important to sign in _before_ you start the trace so that the trace doesn't contain sensitive information related to your account.

1. Start recording the steps you take in the portal by using [Snipping Tool](https://support.microsoft.com/en-us/windows/use-snipping-tool-to-capture-screenshots-00246869-1843-655f-f220-97299b865f6b), [Clipchamp](https://support.microsoft.com/en-us/topic/how-to-make-a-screen-recording-8797f456-7edd-4176-b525-28b954ff5e4d), or another screen recording tool.

1. In the portal, navigate to the step prior to where the issue occurs.

1. Press F12 to launch Microsoft Edge DevTools. You can also launch the tools from the toolbar menu under **More tools** > **Developer tools**.

1. By default, the browser keeps trace information only for the page that's currently loaded. Set the following options so the browser keeps all trace information, even if your repro steps require going to more than one page.

    1. Select the **Console** tab, select **Console settings**, then select **Preserve Log**.

       :::image type="content" source="media/capture-browser-trace/edge-console-preserve-log.png" alt-text="Screenshot that highlights the Preserve log option on the Console tab in Edge.":::

    1. Select the **Network** tab. If that tab isn't visible, click the **More tools** (+) button and select **Network**. Then, from the **Network** tab, select **Preserve log**.

       :::image type="content" source="media/capture-browser-trace/edge-network-preserve-log.png" alt-text="Screenshot that highlights the Preserve log option on the Network tab in Edge.":::

1. On the **Network** tab, select **Stop recording network log** and **Clear**.

    :::image type="content" source="media/capture-browser-trace/edge-stop-clear-session.png" alt-text="Screenshot showing the Stop recording network log and Clear options on the Network tab in Edge.":::

1. Select **Record network log**, then reproduce the issue in the portal.

   :::image type="content" source="media/capture-browser-trace/edge-start-session.png" alt-text="Screenshot showing how to record the network log in Edge.":::

1. After you have reproduced the unexpected portal behavior, select **Stop recording network log** again, then select **Export HAR (sanitized)...** and save the file. If you don't see the **Export HAR** icon, expand the width of your Edge developer tools window.

   :::image type="content" source="media/capture-browser-trace/edge-network-export-har.png" alt-text="Screenshot showing how to Export HAR on the Network tab in Edge.":::

1. Stop recording your screen, and save the video file.

1. Back in the browser developer tools pane, select the **Console** tab. Right-click one of the messages, then select **Save as...**, and save the console output to a text file.

1. Package the browser trace HAR file, console output, and screen recording files in a compressed format such as .zip.

1. Share the compressed file with Microsoft support by [using the **File upload** option in your support request](supportability/how-to-manage-azure-support-request.md#upload-files).

## Google Chrome

The following steps show how to use the developer tools in Google Chrome to capture a browser trace. For more information, see [Chrome DevTools](https://developers.chrome.com/docs/devtools).

1. Sign in to the [Azure portal](https://portal.azure.com). It's important to sign in _before_ you start the trace so that the trace doesn't contain sensitive information related to your account.

1. Start recording the steps you take in the portal. You can use [Snipping Tool](https://support.microsoft.com/en-us/windows/use-snipping-tool-to-capture-screenshots-00246869-1843-655f-f220-97299b865f6b) or [Clipchamp](https://support.microsoft.com/en-us/topic/how-to-make-a-screen-recording-8797f456-7edd-4176-b525-28b954ff5e4d) on Windows, or see [How to record the screen on your Mac](https://support.apple.com/102618).

1. In the portal, navigate to the step prior to where the issue occurs.

1. Press F12 to launch the developer tools. You can also launch the tools from the toolbar menu under **More tools** > **Developer tools**.

1. By default, the browser keeps trace information only for the page that's currently loaded. Set the following options so the browser keeps all trace information, even if your repro steps require going to more than one page:

   1. Select the **Console** tab, select **Console settings**, then select **Preserve Log**.

      :::image type="content" source="media/capture-browser-trace/chrome-console-preserve-log.png" alt-text="Screenshot that highlights the Preserve log option on the Console tab in Chrome.":::

   1. Select the **Network** tab, then select **Preserve log**.

      :::image type="content" source="media/capture-browser-trace/chrome-network-preserve-log.png" alt-text="Screenshot that highlights the Preserve log option on the Network tab in Chrome.":::

1. On the **Network** tab, select **Stop recording network log** and **Clear**.

   :::image type="content" source="media/capture-browser-trace/chrome-stop-clear-session.png" alt-text="Screenshot showing the Stop recording network log and Clear options on the Network tab in Chrome.":::

1. Select **Record network log**, then reproduce the issue in the portal.

   :::image type="content" source="media/capture-browser-trace/chrome-start-session.png" alt-text="Screenshot that shows how to record the network log in Chrome.":::

1. After you have reproduced the unexpected portal behavior, select **Stop recording network log**, then select **Export HAR (sanitized)...** and save the file.

   :::image type="content" source="media/capture-browser-trace/chrome-network-export-har.png" alt-text="Screenshot that shows how to export a HAR file on the Network tab in Chrome.":::

1. Stop recording your screen, and save the video file.

1. Back in the browser developer tools pane, select the **Console** tab. Right-click one of the messages, then select **Save as...**, and save the console output to a text file.

1. Package the browser trace HAR file, console output, and screen recording files in a compressed format such as .zip.

1. Share the compressed file with Microsoft support by [using the **File upload** option in your support request](supportability/how-to-manage-azure-support-request.md#upload-files).

## Safari

The following steps show how to use the developer tools in Apple Safari on Mac. For more information, see [Safari Developer Tools](https://developer.apple.com/safari/tools/).

1. Enable the developer tools in Safari:

   1. Select **Safari**, then select **Preferences**.

   1. Select the **Advanced** tab, then select **Show Develop menu in menu bar**.

      :::image type="content" source="media/capture-browser-trace/safari-show-develop-menu.png" alt-text="Screenshot of the Safari Advanced Preferences options.":::

1. Sign in to the [Azure portal](https://portal.azure.com). It's important to sign in _before_ you start the trace so that the trace doesn't contain sensitive information related to your account.

1. Start recording the steps you take in the portal. For more information, see [How to record the screen on your Mac](https://support.apple.com/HT208721).

1. In the portal, navigate to the step prior to where the issue occurs.

1. Select **Develop**, then select **Show Web Inspector**.

   :::image type="content" source="media/capture-browser-trace/safari-show-web-inspector.png" alt-text="Screenshot of the Show Web Inspector option on the Safari Develop menu.":::

1. By default, the browser keeps trace information only for the page that's currently loaded. Set the following options so the browser keeps all trace information, even if your repro steps require going to more than one page:

    1. Select the **Console** tab, then select **Preserve Log**.

       :::image type="content" source="media/capture-browser-trace/safari-console-preserve-log.png" alt-text="Screenshot of the Preserve Log option in the Console tab on Safari.":::

    1. Select the **Network** tab, then select **Preserve Log**.

       :::image type="content" source="media/capture-browser-trace/safari-network-preserve-log.png" alt-text="Screenshot of the Preserve Log option in the Network tab on Safari.":::

1. On the **Network** tab, select **Clear Network Items**.

   :::image type="content" source="media/capture-browser-trace/safari-clear-session.png" alt-text="Screenshot of the Clear Network Items option in the Network tab on Safari.":::

1. Reproduce the issue in the portal.

1. After you have reproduced the unexpected portal behavior, select **Export** and save the file.

   :::image type="content" source="media/capture-browser-trace/safari-network-export-har.png" alt-text="Screenshot of the Export command in the Network tab on Safari.":::

1. Stop recording your screen, and save the video file.

1. Back in the browser developer tools pane, select the **Console** tab, and expand the window. Place your cursor at the start of the console output then drag and select the entire contents of the output. Use Command-C to copy the output and save it to a text file.

1. Package the browser trace HAR file, console output, and screen recording files in a compressed format such as .zip.

1. Share the compressed file with Microsoft support by [using the **File upload** option in your support request](supportability/how-to-manage-azure-support-request.md#upload-files).

## Firefox

The following steps show how to use the developer tools in Firefox. For more information, see [Firefox DevTools User Docs](https://firefox-source-docs.mozilla.org/devtools-user/index.html).

1. Sign in to the [Azure portal](https://portal.azure.com). It's important to sign in _before_ you start the trace so that the trace doesn't contain sensitive information related to your account.

1. Start recording the steps you take in the portal. You can use [Snipping Tool](https://support.microsoft.com/en-us/windows/use-snipping-tool-to-capture-screenshots-00246869-1843-655f-f220-97299b865f6b) or [Clipchamp](https://support.microsoft.com/en-us/topic/how-to-make-a-screen-recording-8797f456-7edd-4176-b525-28b954ff5e4d) on Windows, or see [How to record the screen on your Mac](https://support.apple.com/102618).

1. In the portal, navigate to the step prior to where the issue occurs.

1. Press F12 to launch the developer tools. You can also launch the tools from the toolbar menu under **More tools** > **Web developer tools**.

1. By default, the browser keeps trace information only for the page that's currently loaded. Set the following options so the browser keeps all trace information, even if your repro steps require going to more than one page:

    1. Select the **Console** tab, select the **Settings** icon, and then select **Persist Logs**.

       :::image type="content" source="media/capture-browser-trace/firefox-console-persist-logs.png" alt-text="Screenshot of the Console setting for Persist Logs in Firefox":::

    1. Select the **Network** tab, select the **Settings** icon, and then select **Persist Logs**.

       :::image type="content" source="media/capture-browser-trace/firefox-network-persist-logs.png" alt-text="Screenshot of the Network setting for Persist Logs in Firefox.":::

1. On the **Network** tab, select **Clear**.

    :::image type="content" source="media/capture-browser-trace/firefox-clear-session.png" alt-text="Screenshot of the Clear option on the Network tab in Firefox.":::

1. Reproduce the issue in the portal.

1. After you have reproduced the unexpected portal behavior, select **Save All As HAR**.

   :::image type="content" source="media/capture-browser-trace/firefox-network-export-har.png" alt-text="Screenshot of the Save All As HAR command on the Network tab in Firefox.":::

1. Stop recording your screen, and save the video file.

1. Back in the browser developer tools pane, select the **Console** tab. Right-click one of the messages, then select **Save All Messages to File**, and save the console output to a text file.

1. Package the browser trace HAR file, console output, and screen recording files in a compressed format such as .zip.

1. Share the compressed file with Microsoft support by [using the **File upload** option in your support request](supportability/how-to-manage-azure-support-request.md).

## Next steps

- Read more about the [Azure portal](azure-portal-overview.md).
- Learn how to [open a support request](supportability/how-to-create-azure-support-request.md) in the Azure portal.
- Learn more about [file upload requirements for support requests](supportability/how-to-manage-azure-support-request.md).
