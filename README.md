# Introduction [Early draft]

This is a quality of life improvement idea for passwordmanagers and websites.
The idea is to have a standard for passwordmanagers to automatically change passwords for accounts on websites.
It consists of a manifest file hosted by websites with user accounts who want to support this feature.
The Manifest file contains information on how to change the password for the account and how to delete the account.
Also instructions for automatic requests from passwordmanagers are included.
The passwordmanager can then use this information to automatically change the password for the account or delete the account.
Or at least prepare everything for the user to do it with just a few clicks.

I got the idea at night and it might even already exist somewhere.

## Use cases

1. A user wants to change their password on a website.
   - After a data breach.
   - Regularly changing passwords.
   - Switching repeatedly used passwords for unique passwords.

2. A user wants to delete their account on a website.
    - After quitting a service or just because they want to delete their account.
    - Enterprises want to delete the accounts of employees who leave the company.
    - A Deceased users family wants to delete their accounts.

## Manifest file structure

```json
{
    "version": 1,
    "allowInsecure (Optional for debug to allow http)": true | false,
    "[Type of action] (changePassword, deleteAccount, other)": {
        "url": "[Endpoint for action]",
        "method": "[HTTP method for action]",
        "authentification": "token" | "header" | "password" | "none",
        "headers (Optional)": {
            "[Header name]": "[Header value]"
        },
        "body (Optional)": "[Body for action]",
        "success (Optional)": {
            "status": "[HTTP status code for success]",
            "body (Optional)": "[Body for success]"
            "type": "contains" | "equals" | "regex" | "none",
            "callback (Optional)": "[Endpoint for callback]",
        },
        "interactivity (Optional)": "none" | "mail_confirm" | "mail_confirm_staged" | "2fa" | "other",
        "interactivityEndpoint (Optional)": "[Endpoint for interactivity]",
        "interactivityHeaders (Optional)": {
            "[Header name]": "[Header value]"
        },
        "interactivityBody (Optional)": "[Body for interactivity]",
        "interactivityCallback (Optional)": "periodic" | "long_polling" | "none",
        "interactivityCallbackHeaders (Optional)": {
            "[Header name]": "[Header value]"
        },
        "interactivityCallbackBody (Optional)": "[Body for interactivity callback]",
        "interactivityInstructions (Optional)": "[Instructions for interactivity]",
        "interactivitySuccess (Optional)": {
            "status": "[HTTP status code for success]",
            "body (Optional)": "[Body for success]"
            "type": "contains" | "equals" | "regex" | "none",
        },
        "interactivityFailure (Optional)": {
            "redirect": "[URL to redirect to/open in browser]",
            "message (Optional)": "[Message to display to user]",
            "restart": "complete" | "interactivity" | "none",
        },
    }
```

## Special values

1. {2FA} - The 2FA code from the authenticator app
   - Requests a 2FA code from the user, a passwordmanager should display a dialog to enter the code or fill it in automatically if it already has it.
2. {USER_INPUT} - Userinput from the passwordmanager
   - Requests a user input, this could be a SMS code, a captcha or a security question.

## Interactivity

Interactivity is a way to interact with the user during the process.
This could be a captcha, a security question or a 2FA code.
The passwordmanager should display a dialog to the user to enter the information.

### Interactivity types

1. none - No interactivity
2. mail_confirm - Confirm the change via email
   - The user gets an email with a link to confirm the change.
3. mail_confirm_staged - Confirm the change via email in two steps
   - The user gets an email with a link to confirm the change, but the change is not yet applied.

> [!WARNING]
> Havent thought about mail_confirm_staged should look like yet. But it is intended for cases where the user has to confirm the change before sending the new password.

### Interactivity callback

The interactivity callback is a way for the website to notify the passwordmanager that the interactivity is done.
This could be a periodic request or a long polling request.
The passwordmanager should display a dialog to the user to wait for the interactivity to be done.
This can also be a toast notification or something similar.

### Interactivity failure

The interactivity failure is a way for the website to notify the passwordmanager that the interactivity failed.
This could be a captcha that was entered wrong or a security question that was answered wrong.
The passwordmanager should display a dialog to the user to notify them that the interactivity failed.
This can also be a toast notification or something similar.

### Interactivity restart

The interactivity restart is a way for the website to notify the passwordmanager that the interactivity failed and should be restarted.
It tells the passwordmanager what to restart.
This could be the whole process or just the interactivity part.

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
    "changePassword": {
        "url": "https://example.com/changePassword",
        "method": "POST",
        "authentification": "password",
        "body": {
            "oldPassword": "[Old password]",
            "newPassword": "[New password]",
        },
        "success": {
            "status": 200,
            "body": "Password change started successfully",
            "type": "contains",
            "callback": "https://example.com/changePasswordStatus?token=1234567890"
        },
        "interactivity": "2fa",
        "interactivityCallback": "long_polling",
        "interactivityCallbackHeaders":  {
            "2fa": "{2FA}",
            "userInput": "{USER_INPUT}",
        },
        "interactivityInstructions": "Please enter the 2FA code from your authenticator app. Security question: What is your favorite color?",
        "interactivitySuccess": {
            "status": 200,
            "body": "Password change confirmed successfully",
            "type": "contains",
        },
        "interactivityFailure": {
            "redirect": "https://example.com/faq/changePassword",
            "message": "Password change not confirmed. Did you enter the correct 2FA code and answer the security question correctly?",
        },
    },
```

## Contributing

I dont know how to contribute to this yet.
It is just an idea I had and I dont know if it already exists somewhere or in some form.
If you know something similar or have ideas on how to improve this please let me know.
If you want to contribute just open an issue or a pull request and i might look at it.
To use this idea you dont need to credit me or anything, just use it. I would love to see something like this in the future.
