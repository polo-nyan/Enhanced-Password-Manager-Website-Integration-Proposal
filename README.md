# Introduction [Early draft]

This suggestion aims to enhance the quality of life regarding password managers and websites.
The proposal suggests establishing a standardized process for password managers to automatically update passwords for accounts on various websites.
This involves the creation of a manifest file, which is hosted by websites whose user accounts wish to support this functionality.

Within the manifest file, crucial information is provided, including instructions on changing the account password and deleting the account. Additionally, the file contains guidelines for facilitating automatic requests from password managers.
By utilizing this information, password managers can seamlessly execute tasks such as password updates or account deletions automatically. Alternatively, the system can be designed to streamline the process for users, requiring only a few clicks to complete the necessary actions.

This idea occurred to me during the night, and there is a possibility that a similar concept may already exist somewhere.

## Use cases

1. A user wants to change their password on multiple websites.
   - After a data breach.
   - Regularly changing passwords.
   - Switching repeatedly used passwords for unique passwords.

2. A user wants to delete their account on multiple websites.
    - After quitting a service or just because they want to delete their account.
    - Enterprises want to delete the accounts of employees who leave the company.
    - A Deceased users family wants to delete their accounts.

3. Automatic Password rotation for security reasons.
    - Enterprises want to rotate shared passwords for platforms like social media.

4. Passwordmanagers could rotate a Users Password immidiatly after a Data breach becomes public.
    - This could be automated by using HaveIBeenPwned or other services.
    - Triggering a password change for all accounts that are affected by the breach. (Triggered by the manifest, a psuh notification from the website or a service like HaveIBeenPwned)

## Manifest file structure

```json
{
    "version": 1,
    "allowInsecure (Optional for debug to allow http)": true | false,
    "sessionName": "[Name of session cookie]",
    "[Type of action] (changePassword, deleteAccount, other)": {
        "endpoint": "[Endpoint for action]",
        "method": "[HTTP method for action]",        
        "authentification": "token" | "header" | "password" | "none",
        "header (Optional)": {
            "[Header name]": "[Header value]",
            "pma (Optional)": "token" | "Certificate" | "none",
        },
        "body (Optional)": "[Body for action]",
        "success (Optional)": {
            "status": "[HTTP status code for success]",
            "body (Optional)": "[Body for success]"
            "type": "contains" | "equals" | "regex" | "none",
            "callback (Optional)": "[Endpoint for callback]",
        },
        "failure (Optional)": {
            "redirect (Optional)": "[URL to redirect to/open in browser]",
            "message (Optional)": "[Message to display to user]",
            "restart": "complete" | "interactivity" | "none",
        },
        "interactivity": {
            "type ": "none" | "mail_confirm" | "mail_confirm_staged" | "2fa" | "other",
            "endpoint (Optional)": "[Endpoint for interactivity]",
            "header (Optional)": {
                "[Header name]": "[Header value]"
            },
            "body (Optional)": "[Body for interactivity]",
            "callback (Optional)": "periodic" | "long_polling" | "none",
            "callbackInterval (Optional)": "[Interval in seconds for periodic callback in seconds if periodic is used]",
            "callbackHeader (Optional)": {
                "[Header name]": "[Header value]"
            },
            "callbackBody (Optional)": "[Body for interactivity callback]",
            "instructions (Optional)": "[Instructions for interactivity]",
            "success (Optional)": {
                "status": "[HTTP status code for success]",
                "body (Optional)": "[Body for success]"
                "type": "contains" | "equals" | "regex" | "none",
            },
            "failure (Optional)": {
                "redirect (Optional)": "[URL to redirect to/open in browser]",
                "message (Optional)": "[Message to display to user]",
                "restart": "complete" | "interactivity" | "none",
            },
        }
    }
}
```

## Special values

1. {2FA} - The 2FA code from the authenticator app
   - Requests a 2FA code from the user, a passwordmanager should display a dialog to enter the code or fill it in automatically if it already has it.
2. {USER_INPUT} - Userinput from the passwordmanager
   - Requests a user input, this could be a SMS code, a captcha or a security question.
3. {SESSION_TOKEN} - A session token from the website
   - Can be set by the website to allow the passwordmanager to use the session of the user to change the password.
   - Useful for Callbacks that require authentication.
   - After the first request the passwordmanager should store the session token and use it for all future requests.
  
## Interactivity

The interactive aspect of the process is facilitated through the interactivity section of the manifest file. This can encompass elements such as a captcha, a security question, or a 2FA code. The password manager is expected to present a user-friendly dialog prompting the user to input the required information. This ensures an additional layer of security and user verification during the process.

### Interactivity types

1. none - No interactivity
2. mail_confirm - Confirm the change via email
   - The user gets an email with a link to confirm the change.
3. mail_confirm_staged - Confirm the change via email in two steps
   - The user gets an email with a link to confirm the change, but the change is not yet applied.

> [!WARNING]
> Havent thought about mail_confirm_staged should look like yet. But it is intended for cases where the user has to confirm the change before sending the new password.

### Interactivity callback

The interactivity callback serves as a means for the website to signal the password manager that the interactive phase has concluded. This callback mechanism could take the form of a periodic request or a long polling request. To keep users informed, the password manager is responsible for displaying a dialog, prompting them to wait for the completion of the interactivity process. Additionally, alternative options such as a toast notification or a similar notification mechanism can be employed for a seamless user experience.

### Interactivity failure

The interactivity failure serves as a method for the website to inform the password manager that the interactive process encountered an issue.
This could result from an incorrectly entered captcha or an inaccurate response to a security question.
In response, the password manager should display a user-friendly dialog to notify them about the interactivity failure.
Additionally, alternative communication methods, such as a toast notification or a similar interface element, can be employed for effectively conveying this information to the user.

### Interactivity restart

The interactivity restart function enables the website to communicate to the password manager that the previous interactivity attempt failed and needs to be restarted.
This notification includes instructions on what specific aspect of the process requires a restart, whether it be the entire process or just the interactivity segment.
By providing this information, the password manager can implement the necessary actions to restart the indicated components, ensuring a smoother and more resilient interactive experience for the user.

## Actions

### Change password

The change password action is used to change the password for an account.

### Delete account

The delete account action is used to delete an account.
This can be useful in case of a deceased user or if the user wants to delete their accounts in a lot of places.
Or if a user is part of an enterprise and leaves the company. The company could use this to delete the account of the employee in all the services they used.

### Other

Non standard actions can be used for other actions like changing the email address or other things.
This would likely be at the descretion of the website and only usefull for interlocking websites with passwordmanagers.
Enterprises could use this to change the email address of an employee in all the services they use.

## Example Manifest file

```json
{
    "version": 1,
    "allowInsecure": false,
    "sessionName": "session",
    "changePassword": {
        "endpoint": "https://api.example.com/changePassword",
        "method": "POST",
        "authentification": "token",
        "header": {
            "pma": "token123456789",
        },
        "body": "{\"password\":\"newPassword\", \"oldPassword\":\"oldPassword\"}",
        "success": {
            "status": "200",
            "body": "{\"message\":\"Password started successfully\"}",
            "type": "contains",
            "callback": "https://api.example.com/callback"
        },
        "failure": {
            "message": "Failed to change password",
            "restart": "none",
        },
        "interactivity": {
            "type": "none",
            "endpoint": "https://api.example.com/interactivity",
            "header": {
                "Session": "{SESSION_TOKEN}"
            },
            "body": "{\"interactivity\":\"none\"}",
            "callback": "periodic",
            "callbackInterval": 5,
            "callbackHeader": {
                "Session": "{SESSION_TOKEN}"
            },
            "callbackBody": "{\"interactivity\":\"none\"}",
            "instructions": "Follow the instructions sent to your email",
            "success": {
                "status": "200",
                "body": "{\"message\":\"Interactiv stage completed successfully\"}",
                "type": "contains",
            },
            "failure": {
                "message": "Failed to complete interactivity",
                "restart": "none",
            },
        }
    }
}
```

## Abusability

A Malicious Actor could abuse this system to immidiatly change the password of a user after they entered their password into a phishing website.
Automated password changes could be used to lock a user out of their account. This could be dangerous after a big data breach.

### Possible Solutions

1. Require 2FA for all changes
   - This would make it harder for a malicious actor to change the password of a user.
2. Allow a X Day rolback usable from a mail containing a link to rollback the change.
   - This would allow a user to rollback the change if they did not initiate it.
   - A Secure Token would be required to rollback the change.
   - This would require the user to have access to their email account.
3. Have some sort of "Signed" Password managers
   - Could be facillitated by "Registering" a Certificate or Token with the website.
   - This would allow the website to verify that the password manager is legitimate.
   - - This could be used to allow password managers to change the password without 2FA.
   - This would require the user to setup the password manager with the website prior to using it.
   - "pma" header (name is subject to change) might be used for this.

## Issues

1. An missmatch of the stored password and the password from the service is possible if the connection is interruped.
   - This could be solved by having a endpoint that checks the password stored and returns if its valid or not.
   - - THIS WOULD BE VERY ABUSABLE
   - The passwordmanager could store the old/new password and the user can rollback the change if the password is wrong.
2. Long Polling might be bad for the servers performance, a timeout defined in the manifest could reduce this.
   - Long Polling should probably not be used for this.
  
## Info

Nothing is spellchecked, checked for grammar, checked for correctness or coherence.
This is just a rough draft of an idea I had and I wanted to write it down before I forget it.

## Contributing

I dont know how to contribute to this yet.
It is just an idea I had and I dont know if it already exists somewhere or in some form.
If you know something similar or have ideas on how to improve this please let me know.
If you want to contribute just open an issue or a pull request and i might look at it.
To use this idea you dont need to credit me or anything, just use it. I would love to see something like this in the future.
