---
description: >-
  In this section you will find the different codes and values that we return in
  several of our services.
---

# Result Codes and References

### ageEstimateGroup

0 = NOT\_AVAILABLE, 1 = UNDER8, 2 = OVER8, 3 = OVER13, 4 = OVER18, 5 = OVER21, 6 = OVER25, 7 = OVER30



### Verification matchLevel FaceMap3D:FaceMap3D

When the process executed is a verification between faces captured with our technology where we extract the 3D face map and compare it against another one stored in our database, the results can be:

Unicus Match Levels:

* Match Level 15 - 1/125,000,000 FAR
* Match Level 14 - 1/95,000,000 FAR
* Match Level 13 - 1/70,00,000 FAR
* Match Level 12 - 1/50,000,000 FAR
* Match Level 11 - 1/25,000,000 FAR
* Match Level 10 - 1/12,800,000 FAR
* Match Level 9 - 1/2,000,000 FAR
* Match Level 8 - 1/1,000,000 FAR
* Match Level 7 - 1/500,000 FAR
* Match Level 6 - 1/100,000 FAR
* Match Level 5 - 1/10,000 FAR
* Match Level 4 - 1/1,000 FAR
* Match Level 3 - 1/500 FAR
* Match Level 2 - 1/250 FAR
* Match Level 1 - 1/100 FAR
* Match Level 0 - Non-match



### Enrollment matchLevel FaceMap3d:Photo ID

In the enrollment process, when a match is made between the person's face and the photo contained in his/her ID card, the match results can be:

Unicus 3D:2D Photo Match Levels & associated accuracy:

* Match Level 7 - 1/500,000 FAR
* Match Level 6 - 1/100,000 FAR
* Match Level 5 - 1/10,000 FAR
* Match Level 4 - 1/1,000 FAR
* Match Level 3 - 1/500 FAR
* Match Level 2 - 1/250 FAR
* Match Level 1 - 1/100 FAR
* Match Level 0 - Non-match

\* The False Reject Rates vary with types of Anti-tampering on IDs and age of the Photo on ID.



### Event response codes <a href="#event-codes" id="event-codes"></a>

<table><thead><tr><th width="176">Error code</th><th>Description</th></tr></thead><tbody><tr><td>500</td><td>General Error</td></tr><tr><td>1001</td><td>Invalid document verification</td></tr><tr><td>1004</td><td>Server error</td></tr><tr><td>1005</td><td>Communication error</td></tr><tr><td>2001</td><td>Session not generated</td></tr><tr><td>2002</td><td>Configuration not generated</td></tr><tr><td>2011</td><td>User already registered</td></tr><tr><td>2012</td><td>User not registered</td></tr><tr><td>2021</td><td>User already verified</td></tr><tr><td>2022</td><td>User not verified</td></tr><tr><td>2023</td><td>User not found</td></tr><tr><td>2031</td><td>ID not Found</td></tr><tr><td>2041</td><td>Process canceled by user</td></tr><tr><td>2051</td><td>Transaction not found</td></tr><tr><td>2052</td><td>User is currently blocked</td></tr><tr><td>2061</td><td>Process cancelled due to retries</td></tr><tr><td>3001</td><td>Registered user</td></tr><tr><td>3002</td><td>User not registered</td></tr><tr><td>4001</td><td>Process canceled by user</td></tr><tr><td>5001</td><td>Problem saving selfie</td></tr><tr><td>5002</td><td>Problem saving audit image</td></tr><tr><td>5003</td><td>Liveness could not be determined</td></tr><tr><td>5004</td><td>Internal selfie server error</td></tr><tr><td>5005</td><td>Communication error selfie</td></tr><tr><td>6001</td><td>Invalid front document</td></tr><tr><td>6002</td><td>Invalid back document</td></tr><tr><td>6003</td><td>Timeout</td></tr><tr><td>6004</td><td>Document not supported</td></tr><tr><td>6005</td><td>Communication error</td></tr><tr><td>6006</td><td>Document does not match</td></tr><tr><td>6007</td><td>Error BASE64</td></tr><tr><td>6008</td><td>Document not loaded in the system</td></tr><tr><td>6009</td><td>Invalid document material</td></tr><tr><td>7001</td><td>Low quality front document</td></tr><tr><td>7002</td><td>Low quality back document</td></tr><tr><td>7003</td><td>Error time out OCR</td></tr><tr><td>7004</td><td>Internal OCR service error</td></tr><tr><td>7005</td><td>OCR communication error</td></tr><tr><td>7006</td><td>Document text could not be read</td></tr><tr><td>7007</td><td>Failed document</td></tr><tr><td>7008</td><td>Document not available</td></tr><tr><td>8001</td><td>User not found</td></tr><tr><td>8002</td><td>User not verified by government</td></tr><tr><td>8003</td><td>Error response time government service</td></tr><tr><td>8004</td><td>Internal Government Error</td></tr><tr><td>8005</td><td>Communication error Government</td></tr><tr><td>8006</td><td>Government not available</td></tr><tr><td>9001</td><td>Your photo did not match your document</td></tr><tr><td>9002</td><td>Did not match his face</td></tr><tr><td>9004</td><td>Internal error match</td></tr><tr><td>9005</td><td>Error communication match</td></tr></tbody></table>
