# Sending email

Although Python provides a mail sending interface via the [`smtplib`](https://docs.python.org/3/library/smtplib.html#module-smtplib) module, Django provides a couple of light wrappers over it. These wrappers are provided to make sending email extra quick, to help test email sending during development, and to provide support for platforms that can't use SMTP.