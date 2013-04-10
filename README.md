# tus resumable upload protocol

**Version:** 0.1 ([SemVer](http://semver))

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be
interpreted as described in [RFC 2119](http://www.ietf.org/rfc/rfc2119.txt).

## Status

Interested developers are encouraged to implement prototypes of this protocol
and submit their feedback in form of comments of patches.

## Abstract

The tus resumable upload protocol describes a light-weigh mechanism for file
uploads over http that can be resumed in the event of a network failure.

Features of the protocol include:

* Uploading files via HTTP
* Resuming interrupted uploads
* Transferring parts of a file in parallel

## Protocol

This protocol is defined as subset of
[RFC 2616](http://tools.ietf.org/html/rfc2616) (HTTP 1.1) to provide a
[RESTful](http://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm)
mechanism for resumable file uploads.

## Error Handling

Servers MUST use appropriate http status codes for all error responses. In
particular, they SHOULD use `400 Bad Request` for invalid requests, `501 Not
Implemented` for unsupported features and `500 Internal Error` for internal
problems.

Both clients and servers SHOULD attempt to detect and handle network errors
predictably. They may do so by checking for read/write socket errors, as well
as setting read/write timeouts. Both clients and servers SHOULD use a 30 second
timeout . A timeout SHOULD be handled by closing the underlaying socket.

Servers SHOULD always attempt to process partial messages bodies in order to
store as much of the received data as possible.

Clients SHOULD use a randomized exponential back off strategy before repeating
requests or resuming uploads interrupted due to network errors as well as after
receiving a `500 Internal Error`. It is up to the client to decide to give up
at some point.

### Creating file resources

A server SHOULD define one or more fixed URLs for clients to create new file
resources via `POST` requests (e.g `/files`).  A client MAY indicate the
desired location for the new resource (e.g. `/files/sha1sum`), but MUST be
prepared to receive a different `Location` header in the server response.

All file resource creation requests MUST include a `Content-Range` and
`Content-Length` header.

The `Content-Length` defines the amount of bytes included in the request body.
For resumable uploads it SHOULD be set to `0`, but clients MAY choose to upload
some or all bytes of a file when creating it.

The `Content-Range` defines the total size of the file, and optionally the data
range included in the body of the request. When `Content-Length` is `0`, the
`Content-Range` MUST be given as `bytes */100` where `100` is the total size of
the file. For `Content-Length` values larger than `0`, the `Content-Range` MUST
take the form `bytes 0-2/100` when sending the first `3` bytes of a `100` byte
message. In this case the `Content-Length` would be 3. For simplicity, servers
MAY choose to implement support for ranges beginning at an offset of 0.

A valid request MUST be acknowledged with a `201 Created` status by the server.
The response MUST also include a `Location` header that holds the absolute URL
of the created file resource.

Clients and servers MAY choose to include additional headers for application
specific purposes, such as `Content-Type`, `Content-Disposition`, etc. to
provide meta information or processing directives to the server.

**Request Example:**

The request below creates an empty file resource against a tus endpoint at
`http://tus.example.com/files`.

```
POST /files HTTP/1.1
Host: tus.example.com
Content-Length: 0
Content-Range: bytes */100
```
```
<empty body>
```

**Response Example:**

```
HTTP/1.1 201 Created
Location: http://tus.example.com/files/24e533e02ec3bc40c387f1a0e460e216
Content-Length: 0
```

### PUT &lt;fileUrl&gt;

**Request Example:**

```
PUT /files/24e533e02ec3bc40c387f1a0e460e216 HTTP/1.1
Host: tus.example.com
Content-Length: 100
Content-Range: bytes 0-99/100
```
```
<bytes 0-99>
```

**Response Example:**

```
HTTP/1.1 200 Ok
Content-Type: image/jpg
Content-Disposition: attachment; filename="me.jpg"'
Range: bytes=0-99
Content-Length: 0
```

### HEAD &lt;fileUrl&gt;

**Request Example:**

```
HEAD /files/24e533e02ec3bc40c387f1a0e460e216 HTTP/1.1
Host: tus.example.com
```

**Response Example:**

```
HTTP/1.1 200 Ok
Content-Length: 100
Content-Type: image/jpg
Content-Disposition: attachment; filename="me.jpg"'
Range: bytes=0-20,40-99
```

The `Range` header holds a [byte
range](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.35.1) that
informs the client which parts of the file have been received so far. It is
up to the client to choose appropriate `PUT` requests to complete the upload.

A completed upload will be indicated by a single range covering the entire file
size (e.g. `Range: bytes=0-99` for a 100 byte file).

**Note** If the server has not received anything so far, there will be no `Range`
header present.

### GET &lt;fileUrl&gt;

Used to download an uploaded file.

**Request:**

```
GET /files/24e533e02ec3bc40c387f1a0e460e216 HTTP/1.1
Host: tus.example.com
```

**Response:**

```
HTTP/1.1 200 Ok
Content-Length: 100
Content-Type: image/jpg
Content-Disposition: attachment; filename="me.jpg"'
```
```
[file data]
```

## Appendix A - Discussion of Prior Art

*to be written ...*

**Prior art:**

* [YouTube Data API - Resumable Upload](https://developers.google.com/youtube/v3/guides/using_resumable_upload_protocol)
* [Google Drive - Upload Files](https://developers.google.com/drive/manage-uploads)
* [Resumable Media Uploads in the Google Data Protocol](https://developers.google.com/gdata/docs/resumable_upload) (deprecated)
* [ResumableHttpRequestsProposal from Gears](http://code.google.com/p/gears/wiki/ResumableHttpRequestsProposal) (deprecated)

## Appendix B - Cross Domain Uploads

*to be written ...*

## Appendix C - Support for legacy / multipart clients

*to be written ...*

## License

Copyright (c) 2013 Transloadit Ltd and Contributors. Licensed under the MIT
license, see
[LICENSE.txt](https://github.com/tus/tus-resumable-upload-protocol/blob/master/LICENSE.txt).

## Contributing

This protocol has it's own [Github repository](https://github.com/tus/tus-resumable-upload-protocol)
where you can leave feedback and pull requests.
