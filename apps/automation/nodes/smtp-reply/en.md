---
node_id: "smtp-reply"
title: "SMTP: Reply Email"
description: "Reply to a received email in its conversation thread."
category: "actions"
subcategory: "Email"
version: "1.0.0"
language: "en"
---

# SMTP: Reply Email

Uses the same SMTP connection settings as SMTP: Send Email.

| Setting | Mapping |
| --- | --- |
| from | Your sending mailbox |
| to | Original `replyTo`, falling back to original `from` |
| messageId | Original `messageId`; never the numeric IMAP `id` |
| subject | Original `subject` (Re: added automatically if missing) |
| references | Original `references` array, optional |
| text / html | Your reply content |

IMAP Trigger and IMAP Fetch now expose these threading fields. Update both packages. Fetch returns a list: select or split the message before mapping fields. Preserve these fields through intermediate Function/If/Wait nodes or reference the original node output.

Sets In-Reply-To and appends the original Message-ID to References. The outgoing email receives a new Message-ID. Thread display ultimately depends on the recipient mail client. Message-ID alone does not retrieve a recipient or subject over SMTP. This action replies to the configured recipient, not reply-all.

Output includes success, messageId, response, accepted and rejected, matching SMTP: Send Email.

[Message configuration](https://nodemailer.com/message)
