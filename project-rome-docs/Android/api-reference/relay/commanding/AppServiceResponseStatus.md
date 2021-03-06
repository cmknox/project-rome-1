---
title: AppServiceResponseStatus 
description: Contains values that describe the status of a message sent from one app service to another.
keywords: microsoft, windows, device relay, Android, Android api reference
---

# AppServiceResponseStatus enum
Contains values that describe the status of a message sent from one app service to another (whether the message data was successfully delivered).


|Name   |Value   |Description   |
|:--------|:-------|:-------------|
|SUCCESS |0 |The message was delivered successfully. |
|FAILURE |1 |The message was not delivered due to network failure. |
|RESOURCE_LIMITS_EXCEEDED |2 |The message was not delivered because it exceeded the program memory limits of the remote app service. |
|UNKNOWN |3 | The messaged was not delivered for an unknown reason.|
|REMOTE_SYSTEM_UNAVAILABLE |4 |The message was not delivered because the connection to the remote device was lost. |
|MESSAGE_SIZE_TOO_LARGE |5 |The message was not delivered because it exceeded the allowed size. |
|APPSERVICE_CONNECTION_CLOSED|6 |The message was not delivered because the connection to the remote app service was closed.

