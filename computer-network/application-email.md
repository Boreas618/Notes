# Email

Four major components:

* Mail User Agent

* Mail Transfer Agent

  * **Mailbox** contains incoming messages for user
  * **Message queue** of outgoing (to be sent) mail messages

* SMTP

  Between UA and server/ Between servers

* Mail Access Protocol

  POP3, IMAP, ...

<img src="https://p.ipic.vip/4e3kqd.png" alt="Screenshot 2023-06-13 at 6.50.07 PM" style="zoom:50%;" />

## Mail Format

**Header**

* 7-bit U.S. ASCII printable characters (ASCII code 32-126)
* Multiple lines of text, each line containing a type and value separated by a colon (:)
* **There should be no space before the colon**
* Defines many commonly used email header types (From:, To:, CC:, Reply-To:, Subject:), case-insensitive

**Body**

* 7-bit U.S. ASCII characters (ASCII codes 1-127)
* There is a blank line between the header and the body of the email \<CRLF>\<CRLF> `\r\n\r\n`

## MIME

MIME stands for Multipurpose **I**nternet **M**ail **E**xtension.

### Components

- **MIME Entity**: A MIME construct that has headers and body. The main body of the email is also considered a MIME entity.
- **Multipart Entity**: It contains multiple MIME entities. This helps to attach multiple files or types of data to a single email.

### MIME Headers

- **MIME-Version**: Specifies the version of MIME being used, often set to "1.0".
- **Content-Type**: Describes the type of data in the content (e.g., text/plain; charset=utf-8).
  - Subtypes can specify further details, such as whether it's plain text, an image, audio, video, etc.
- **Content-Transfer-Encoding**: Method to encode data, such as ASCII or Base64.
- **Content-ID**: Unique identifier for the MIME content.
- **Content-Description**: Descriptive information about the content.
- **Content-Disposition**: Specifies if the content should be displayed inline or as an attachment.

### MIME Types

- **Text**: Different formats like plain, enriched, CSS, etc.
- **Image**: Formats such as JPEG, GIF, etc.
- **Audio**: Formats like basic, MP3, etc.
- **Video**: Formats like MPEG, MP4, etc.
- **Application**: Various formats like ZIP, PDF, Word, PowerPoint, etc.
- **Multipart**: Special type that includes multiple MIME entities. Further classified as mixed, alternative, parallel, digest, etc.

### Multipart MIME

- A MIME message can contain multiple parts, each with its own headers and content.
  - `multipart/mixed`: The most common subtype; used when there are different types of data (e.g., a text message and some attached images).
  - `multipart/alternative`: Contains the same content in different formats (e.g., a plain text version and an HTML version of the same email).
  - `multipart/related`: Data of one type that references other data in the same message, like an HTML email that references inline images.
- The "boundary" parameter in the **Content-Type** header indicates the separator used between different parts of the message.

```email
From: Alice <alice@example.com>
To: Bob <bob@example.com>
Subject: Weekend Plans
Date: Fri, 14 Oct 2023 10:00:00 +0000
MIME-Version: 1.0
Content-Type: multipart/mixed; boundary="MAIN_BOUNDARY"

--MAIN_BOUNDARY
Content-Type: multipart/alternative; boundary="ALT_BOUNDARY"

--ALT_BOUNDARY
Content-Type: text/plain; charset="utf-8"
Content-Transfer-Encoding: quoted-printable

Hi Bob,

Are we still on for the weekend? Let me know!

Cheers,
Alice

--ALT_BOUNDARY
Content-Type: text/html; charset="utf-8"
Content-Transfer-Encoding: quoted-printable

<html>
<body>
<p><strong>Hi Bob</strong>,</p>
<p>Are we still on for the <em>weekend</em>? Let me know!</p>
<p>Cheers,<br/>Alice</p>
</body>
</html>

--ALT_BOUNDARY--

--MAIN_BOUNDARY
Content-Type: image/jpeg
Content-Disposition: attachment; filename="weekend.jpg"
Content-Transfer-Encoding: base64

[BASE64_ENCODED_IMAGE_DATA_HERE]

--MAIN_BOUNDARY--
```

### Content Transfer Encoding

**Base64**: Three bytes (24 bits) are grouped into four groups, with each group consisting of six bits. 

The 64 values corresponding to each six bits are represented by the characters A~Z, a~z, 0-9, + and /.
When combining in 24-bit blocks, there may be one or two remaining bytes.

Fill the remaining bits (on the right side) with enough zeros to make up a total of six bits before converting them into printable ASCII characters.

Finally, add multiple equal signs. If there is one remaining byte, add two equal signs; if there are two remaining bytes, add one equal sign.

**Quoted-printable**: Printable characters: ASCII codes from 32 to 126, a total of 95 characters including spaces, numbers, uppercase and lowercase letters, and punctuation marks.

Non-ASCII printable characters are converted into three characters: "=" followed by the ASCII code of the character (in uppercase hexadecimal representation).

**The equal sign (=, ASCII code 0011 1101) is converted to =3D.**

## SMTP

Uses TCP to reliably transfer email message from client(mail server initiating connection) to server over port 25.

Mail is sent from the source to the Mail Transfer Agent (MTA), then forwarded to the target address.

Typical SMTP command sequence:

- Client commands include: EHLO, MAIL FROM, RCPT TO, DATA, QUIT, etc.

- Server responses might include specific status codes and messages. For instance:

  > C: EHLO ubuntu 
  >
  > S: 250-mail 
  >
  > S: 250-AUTH LOGIN PLAIN 
  >
  > S: 250 OK

SMTP uses 7-bit ASCII for its message format.

Status code: 3 digits, indicating the success of the execution. The first digit:

* 2 indicates success.
* 3 indicates success, but Client needs to continue sending, such as the status code received when receiving a DATA request.
* 4 indicates temporary refusal.
* 5 indicates permanent refusal.

There are 3 stages: connect, transfer, and close. Among these:

- During connection
- During transfer, the client sends the data, and the server responds.
- The session is terminated during the close stage.

### SMTP and DNS

If there's no MX record for the domain, the A or AAAA records are used.

The smaller **the value of priority** of the server, the closer the server to the destination.

The mail will only be transfered to a sevrer with **a lower priority value** from the current server, which means that the mail is moving closer to the destination.

### SMTP Commands

1. **HELO domain**: Introduction and identification command.
2. **EHLO domain**: Also an introduction command. It's an extension of the HELO command to identify SMTP capabilities. Used to negotiate extended SMTP functionalities.
3. **MAIL FROM:<addr>**: Indicates the sender's email address.
4. **RCPT TO:<addr>**: Indicates the recipient's email address.
5. **DATA**: Begins the body of the message. Message ends with a period (.) on a new line by itself. This is followed by the server sending a confirmation, usually in the form "250 Ok".
6. **QUIT**: Closes the connection.

### Comparision with HTTP

HTTP is client pull while SMTP is client push.

They both have ASCII command/response interaction and status codes.

In HTTP, each object encapsulated in its own response message while in SMTP multiple objects sent in multipart messages.

SMTP uses persistent connections. SMTP requires message (header&body) to be in 7-bit ASCII. SMTP server uses CRLF.CRLF to determine end of message.

SMTP is the protocol for exchanging e-mail messages.

## POP3

It uses TCP and runs on port 110.

### Commands and Responses

> `S: +OK POP3 server ready`: Server sends this when it's ready.
>
> `C: user alice`: Client sends username "alice".
>
> `S: +OK`: Server acknowledges.
>
> `C: pass hungry`: Client sends password "hungry".
>
> `S: +OK user successfully logged on`: Server acknowledges successful login.
>
> ... (and other commands and responses)

**Key Commands**

- `user`: Specify user name.
- `pass`: Specify password.
- `stat`: Get mailbox status.
- `list`: List messages.
- `retr`: Retrieve specific message.
- `dele`: Delete specific message.
- `top`: View top or header of specific message.
- `uidl`: Display unique identifier for messages.
- `quit`: Delete tagged mails and log off from the server.

**Other Features**

- `CAPA`: Shows server capabilities.
- `STLS`: Start a secure session using TLS/SSL.

In the POP3 protocol, emails are downloaded to the local machine and organized locally. They can also be kept on the server, with UIDL used to track which ones have been downloaded locally. Users manage their downloaded emails independently on different machines using a POP3 client.

Typically, all parts of an email are downloaded (later, partial content viewing can be achieved using TOP). The Internet Mail Access Protocol [RFC 3501] is a text-based request/response protocol that uses TCP ports 143 and 993 (IMAP over TLS).

With IMAP, emails are stored and managed by the server. Users can create folders to organize their files. It allows unified access to emails from different hosts.

Clients have the option to only download email headers and retrieve the content when opened. Although emails are stored on the server, clients usually cache read and downloaded messages for convenience.