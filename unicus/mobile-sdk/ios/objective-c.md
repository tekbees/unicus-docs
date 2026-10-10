---
description: >-
  Use the Unicus iOS SDK from Objective-C through the UnicusSdkObjC facade:
  configure, start, resume, cancel and read the result.
---

# Objective-C

Apps written in Objective-C use `UnicusSdkObjC`, a facade over the Swift SDK. It
covers the common path: configure, start a verification for a document, resume
by `tid`, cancel and clear the resume data. Installation is the same as for
Swift ([Installation](installation.md)).

Custom UI steps, progress events and API logs are Swift-only. If you need them,
add a small Swift file to your target that uses `UnicusSdk` directly.

## Import

{% code overflow="wrap" %}
```objc
@import UnicusSDK;   // modules enabled (Xcode default)
```
{% endcode %}

## Configure

{% code overflow="wrap" %}
```objc
NSError *error = [UnicusSdkObjC.shared configureWithApiKey:@"<CUSTOMER_TOKEN>" environment:@"dev"];
if (error) {
    // Unknown environment name: nothing was configured.
    NSLog(@"Unicus: %@", error.userInfo[UnicusSdkObjC.errorCodeKey]);   // invalid_configuration
}
```
{% endcode %}

`environment` accepts `dev`, `staging` or `production` (any case). As in Swift,
only `dev` is available today; the others make `start` fail with
`environment_not_available`.

## Start a verification

{% code overflow="wrap" %}
```objc
[UnicusSdkObjC.shared startWithDocumentType:@"ID"
                             documentNumber:documentNumber
                                  presenter:self
                                 completion:^(UnicusResultObjC *result, NSError *error) {
    if (error) {
        // The verification could not run.
        NSString *code = error.userInfo[UnicusSdkObjC.errorCodeKey];
        NSLog(@"Unicus error %@ (%ld)", code, (long)error.code);
        return;
    }
    if (result.success) {
        // Verified: confirm result.tid in your backend.
    } else if (result.resumable) {
        // 2003: start again with the same document, or resume with result.tid.
    } else if (result.retryable) {
        // Temporary failure: offer to try again.
    } else {
        // Not verified: result.outcome, result.resultCode.
    }
}];
```
{% endcode %}

* `documentType`: `ID`, `FD`, `PP` or `DL`. Another value completes with the
  error code `invalid_request`.
* `presenter` may be `nil`: the SDK uses the top-most view controller.
* Exactly one of `result` / `error` is non-nil. The completion runs on the main
  queue.

## Resume by `tid`

{% code overflow="wrap" %}
```objc
[UnicusSdkObjC.shared startExistingTransaction:savedTid
                                     presenter:self
                                    completion:^(UnicusResultObjC *result, NSError *error) {
    // same handling as above
}];
```
{% endcode %}

Automatic resume with the same document works as in Swift (see
[Results and resuming](../results-and-resuming.md)).

## Cancel and logout

{% code overflow="wrap" %}
```objc
[UnicusSdkObjC.shared cancel];            // cancels the active verification
[UnicusSdkObjC.shared clearResumeData];   // on logout
```
{% endcode %}

## `UnicusResultObjC`

| Property | Type | Meaning |
| --- | --- | --- |
| `success` | `BOOL` | Identity verified. |
| `outcome` | `NSString` | `success`, `warning`, `failed`, `canceled`, `error`, `resumable`, `unknown`. |
| `resultCode` | `NSNumber *` (nullable) | Unicus result code. See [Result codes](../result-codes.md). |
| `resultMessage` | `NSString *` (nullable) | Message of the code, for logs. |
| `tid` | `NSString *` (nullable) | Transaction id. |
| `resumable` | `BOOL` | 2003: the transaction is open. |
| `retryable` | `BOOL` | Temporary failure: start again. |
| `dictionary` | `NSDictionary *` | Every field of the Swift result (`steps`, `flowId`, `rejectionReason`…). |

## Errors (`NSError`)

| Part | Value |
| --- | --- |
| `domain` | `com.tekbees.unicus.sdk` (`UnicusSdkObjC.errorDomain`) |
| `code` | Unicus `resultCode` when the server sent one, else `-1`. |
| `userInfo[UnicusSdkObjC.errorCodeKey]` | Stable error code (`not_configured`, `network_error`, `flow_not_supported`…). Route by this. |
| `userInfo[UnicusSdkObjC.httpStatusKey]` | HTTP status, when there is one. |
| `userInfo[@"UnicusResultMessage"]` | Unicus `resultMessage`, when there is one. |
| `localizedDescription` | Developer text; do not show it to end users. |

Every code: [Errors and troubleshooting](../errors-and-troubleshooting.md).

## Other members

| Member | Purpose |
| --- | --- |
| `UnicusSdkObjC.shared.version` | SDK version. |
| `UnicusSdkObjC.shared.isSessionActive` | `YES` while a verification is being prepared or shown. |
| `configureWithApiKey:baseUrl:` | Advanced: explicit API base URL. Prefer `configureWithApiKey:environment:`. |
