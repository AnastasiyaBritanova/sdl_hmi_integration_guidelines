## Policies: AppID Management

### Table of Contents:
1. [Nickname Validation](#1. Nickname Validation:)
* [AppID Management](#2. AppID Management:)
* [Revoked appID](#Revoked)


### 1. Nickname Validation:

###### SDL-PAM-NV-001: Case-insensitivity for appName validation
SDL must be **case-insensetive** when comparing the value of ``"appName"`` received from the application against the value(s) of ``"nicknames"`` section for the corresponding `appID` provided by the application.

###### SDL-PAM-NV-002: Successful nickname validation
_In case_  
the application sends `RegisterAppInterface` request with  
-> the ``"appName"`` value that is listed in this app's specific policies  
-> other valid parameters  
_SDL must:_  
successfully register such application:  
``RegisterAppInterface_response (<applicable resultCode>, success: true)``

!!! Information
1. ``<Applicable resultCode>``s for successful registration:
   * WRONG_LANGUAGE
   * RESUME_FAILED
   * WARNINGS
   * SUCCESS
2. AppName in Policies example:
```json
{
"policy_table": {
  "app_policies": {
    "<appID>": {
      "nicknames": [ "Demo App", "Super App" ]
    }
  }}}}
```

!!!

###### SDL-PAM-NV-003: Failed nickname validation
_In case_  
the application sends `RegisterAppInterface` request with the `"appName"` value that is   
-> not listed in this app's specific policies  
_SDL must:_  
return ``RegisterAppInterface_response (DISALLOWED, success: false)``  

###### SDL-PAM-NV-004: Nickname validation must be done before duplicate name validation
_In case_  
the application sends `RegisterAppInterface` request with the ``"appName"`` value that is   
-> not listed in this app's specific policies  
-> the same as another already-registered application has  
_SDL must:_  
return ``RegisterAppInterface_response (DISALLOWED, success: false)``  

###### SDL-PAM-NV-005: 	Failed nickname validation for ChangeRegistration
_In case_  
the application sends `ChangeRegistration` request with the ``"appName"`` value that is   
-> not listed in this app's specific policies  
_SDL must:_  
return ``ChangeRegistration_response (DISALLOWED, success: false)`` (not unregister the application)

###### SDL-PAM-NV-006: 	Failed nickname validation after Policies Update
_In case_  
-> the application with ``"appName"`` value is successfully registered with SDL   
-> and the Policies Update removes this `"appName"` value from from `"nicknames"` of this app-assigned policies    
_SDL must:_  
unregister this application via `OnAppInterfaceUnregistered (APP_UNAUTHORIZED)` (The application itself is responsible for closing session with SDL)  

### 2. AppID Management:

###### SDL-PAM-AM-001: Case-insensitivity for appID validation
SDL must be **case-insensetive** when comparing the value of ``"appID"`` received within `RegisterAppInterface` against the value(s) of ``"app_policies"`` section.  

###### SDL-PAM-AM-002: Assign "default" policies to the application which appID does not exist in Local PT
_In case_  
the application registers (sends valid `RegisterAppInterface` request) with the `appID` that does not exist in Local Policy Table    
_SDL must:_  
add this application's ``"<appID>"`` to ``"app_policies"`` section of Local PT and assign ``"default"`` permissions: ``"<appID>": "default"``.

###### SDL-PAM-AM-003: Assign existing policies to the application which appID exists in Local PT
_In case_  
the application registers (sends valid `RegisterAppInterface` request) with the `appID` that exists in Local Policy Table    
_SDL must:_  
apply the existing in ``"<appID>"`` from ``"app_policies"`` section of Local PT permissions to this application.

### 3. Revoked appID (NULL policies): <a name="Revoked"></a>

###### SDL-PAM-RA-001: Only Registration is allowed to revoked applications
_In case_  
PolicyTable has ``"<appID>": "null"`` in the Local PolicyTable     
_SDL must:_  
allow only `RegisterAppInterface` request from the application with this appID.

###### SDL-PAM-RA-002: Disallow all non-registration RPCs to revoked applications  
_In case_  
Local PolicyTable has ``"<appID>": "null"``      
and application with this `appID` sends any RPC request (except RegisterAppInterface)
_SDL must:_  
send `RPC_response (resultCode: DISALLOWED, success: false)` to this application

###### SDL-PAM-RA-003: HMILevel of revoked applications  
_In case_  
Local PolicyTable has ``"<appID>": "null"``      
and application with this `appID` registers
_SDL must:_  
keep this application in `NONE` HMILevel only

###### SDL-PAM-RA-004: HMILevel of applications that become revoked after Policies update
_In case_  
an application with `<appID>` is currently registered and in any `HMILevel` other than in `NONE`      
and in result of PTU this `"<appID>"` gets `"null"` policies
_SDL must:_  
send `OnHMIStatus (NONE, MAIN, NOT_AUDIBLE)` to this application.

###### SDL-PAM-RA-005: Notify HMI about application revoke  
_In case_  
an application with `<appID>` is currently registered and in any `HMILevel` other than in `NONE`      
and in result of PTU this `"<appID>"` gets `"null"` policies
_SDL must:_  
send `OnAppPermissionChanged (appRevoked: true, appID)` to HMI
