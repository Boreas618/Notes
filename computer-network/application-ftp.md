# File Transfer Protocol (FTP)

FTP is divided into two connections: the command connection and the data connection.

- Command Connection: Primarily used for sending control messages.
- Data Connection: Used for sending actual file data.

## **FTP Command Connection** (port 21)

- The client initiates a TCP connection to start a session.
- FTP commands are usually textual and consist of verbs.
- Examples include `USER`, `PASS`, `CWD`, `LIST`, `RETR`, `STOR`, and `REST`.
- Typical FTP responses include:
  - `331` Username OK, password required.
  - `230` Login successful.
  - `125` Data connection already open; transfer starting.
  - `425` Can't open data connection.
  - `452` Error writing file.

### **FTP Response Codes**

- `1xx`: Positive preliminary response.
- `2xx`: Positive completion response.
- `3xx`: Positive intermediate response.
- `4xx`: Transient negative completion response.
- `5xx`: Permanent negative completion response.

## **FTP Data Connection (port 20)**

FTP client initiates the connection on port 20. It operates separately from the command connection, which allows it to transfer files.

## FTP Data Connection Modes: Active vs. Passive

1. **Active Mode (PORT)**:
   - The command format is `PORT h1,h2,h3,h4,p1,p2`.
   - It suggests that the client listens on IP address `h1.h2.h3.h4` and port number `p1*256+p2` for a data connection.
   - The FTP server then initiates the connection to this specified port.
2. **Passive Mode (PASV)**:
   - Once the server listens and is ready for a connection, it informs the client about the IP and port details.

## FTP Flow

For both modes, the flow is as follows:
- `CWD src` command is sent by the client.
- `200` response is a positive completion.
- In Active mode, the `PORT` command specifies where the server should connect. In Passive mode, the `PASV` command asks the server where to connect.
- `LIST` command is issued to get a list of files/directories.
- `150` indicates the start of a file transfer.
- Data is then transmitted.
- `226` indicates the completion of the file transfer.