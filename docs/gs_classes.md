# Useful Classes for Google AppScript
Below are classes that are useful for interacting with [Google Workspace API](https://developers.google.com/workspace) using Google's [appscript](https://developers.google.com/apps-script) language

## Emails

```javascript
class Email {
  /* Email 
  
  Basic Email class 
  */
  constructor (
    emailAddresses, // comma separated list of email addresses (string)
    attachments,  
  ) {
    this._emailAddresses = emailAddresses;
    this._attachments = attachments;
    this._subject = "Subject Not Set"
    this._htmlBodyArray = [];
  }

  get _htmlBody() {
    /* Joins the html content of the email.
    
    If not set a generic message to contact administration
    is set.
    */
    if (this._htmlBodyArray.length > 0) {
      return this._htmlBodyArray.join(" ");
    } else {
      return "<p>Message not implemented. Contact admin.</p>"
    }
  }

  sendEmail() {
    /* Sends an Email */
    const email_log = `
      Subject: ${this._subject}
      Body: ${this._htmlBody}
    `;
    console.log(email_log);
    MailApp.sendEmail(
      this._emailAddresses,
      this._subject,
      this._htmlBody,
      {
        htmlBody: this._htmlBody,
        attachments: this._attachments
      }
    )
  }
}
```
This class is meant to be inherited. For example:
```javascript
class ChildEmailClass extends Email {
  constructor(
    emailAddresses,
    attachments,
  ) {
    super(emailAddresses, attachments)
    this._subject = `
      Specify a custom Subject line
    `
    this._htmlBodyArray = [
      `<h1 align="center">Specify the HTML of the Email Body</h1>`,
      `<h3 align="center">Please see the attachment below for more information.</h3>`,
      `<p align="center">This is great</p>`
    ]
  }

}
```
In use:
```javascript
const emailAddresses = `${emailAddress1},${emailAddress2}` // as many as needed to be specified
const email = new ChildEmailClass(
    emailAddresses,
    [file], // as many as need to be specified. See File objects in API
)
```

### References

- [MailApp](https://developers.google.com/apps-script/reference/mail/mail-app)
- [File](https://developers.google.com/apps-script/reference/drive/file)