# DCS configuration notes
## User notification email templates
It is possible to override the default emails that are sent when user accounts are created etc.

The following actions will result in a notification email being sent to the user. 
The format of each email can be overwritten by the given template which must exist in an EmailTemplate
directory immediately under the directory where DCS Web App is installed.

Note that each template must consist of HTML markup that will be included inside a HTML BODY element
(i.e. the template must not include any HTML or HEADER elements, or the BODY element itself). 
The listed variables may be used in the template and will be replaced at runtime.

Event | Template file name | Variables
------|--------------------|----------
DCS user created | UserCreatedTemplate.html.template | %USERNAME%, %PASSWORD%, %SIGNINURL%, %ROLE%, %EMAIL%
DCS user created with SSO | SsoUserCreatedTemplate.html.template | %SSOIDENTITY%, %USERNAME%, %PASSWORD%, %SIGNINURL%, %ROLE%, %EMAIL%
DCS user created with SSO enforced | SsoEnforcedUserCreatedTemplate.html.template | %SSOIDENTITY%, %SIGNINURL%, %ROLE%, %EMAIL%
User resets their password * | PasswordResetTemplate.html.template | %USERNAME%, %PASSWORD%, %SIGNINURL%, %EMAIL%
DCS account resets by administrator * | AccountResetTemplate.html.template | %USERNAME%, %PASSWORD%, %SIGNINURL%, %EMAIL%
SSO settings for user changed by administrator - SSO disabled | SsoSettingsChangedOffTemplate.html.template | %USERNAME%, %PASSWORD%, %SIGNINURL%, %EMAIL%
SSO settings for user changed by administrator - SSO enabled | SsoSettingsChangedOnTemplate.html.template | %SSOIDENTITY%, %USERNAME%, %PASSWORD%, %SIGNINURL%, %EMAIL%
SSO settings for user changed by administrator - SSO enabled/enforced| SsoSettingsChangedEnforcedTemplate.html.template | %SSOIDENTITY%, %SIGNINURL%, %EMAIL%

> * These actions are not allowed for users with SSO enforced

Variable | Description
---------|------------
%USERNAME% | The DCS account user name
%PASSWORD% | The DCS account password. When this is sent out it will always be a new randomaly generated password.
%SIGNINURL% | Main URL page for DCS. This corresponds to the DcsWebAppUrl settings in the Dcs Web App appSettings.json file. 
%ROLE% | The DCS account role, i.e. Administrator, Guest, Operator or Viewer.
%EMAIL% | The user email associated with the DCS account
%SSOIDENTITY% | The SSO identity associated with the DCS account

A sample template:

**UserCreatedTemplate.html.template**
```
<h1>Your new DCS account has been created</h1>
<p>Your user name is: %USERNAME%</p>
<p>Your password is: %PASSWORD%</p>
<p>The account has been create with the role: %ROLE%</p>
<p>Sign in at:  %SIGNINURL%</p>
```


## Single sign on configuration

By default SSO is completely disabled in DCS. 
The following settings in the DCS Web App appSettings.json file are use to configure SSO support:

Settings | Use | Default value 
---------|-----|--------------
SsoEnabled | Enable SSO support | false
SsoSignInPrompt | UI text used to prompt users to sign in with their SSO account | "Sign in with your SSO account"
SsoLocalSignInPrompt | UI text used to prompt users to signed in with their DCS account | "Sign in with your DCS account"
SsoNotRegisterdErrorTemplate | Error message to user if an attempt is made to sign in with an SSO account that is not registered with any DCS user | "The SSO account %NAME% is not registered for use with DCS"
SsoEnforcedLoginErrorMessage | Error message to user if an attempt is made to sign in with a DCS account for a user that has SSO enforced | "You must use your SSO account to sign into DCS"  
SsoEnforcedErrorMessageAdmin | Error message to administrator when an attempt is made to perform an unallowed action on an account with SSO enforced (viz., Reset) | "This action is not allowed since SSO is enforced on this account"
SsoEnforcedErrorMessageUser | Error message to administrator when an attempt is made to perform an unallowed action on an account with SSO enforced (viz., Reset, Change password) | "You can not perform this action because you must use your SSO account to sign into DCS"



