# Introduction [Early draft]

This suggestion aims to enhance the quality of life regarding password managers and websites. The proposal suggests establishing a standardized process for password managers to automatically update passwords for accounts on various websites. This involves the creation of a manifest file, which is hosted by websites whose user accounts wish to support this functionality.

Within the manifest file, crucial information is provided, including instructions on changing the account password and deleting the account. Additionally, the file contains guidelines for facilitating automatic requests from password managers. By utilizing this information, password managers can seamlessly execute tasks such as password updates or account deletions automatically. Alternatively, the system can be designed to streamline the process for users, requiring only a few clicks to complete the necessary actions.

This idea occurred to me during the night, and there is a possibility that a similar concept may already exist somewhere.

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
