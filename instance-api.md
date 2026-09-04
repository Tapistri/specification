NOTES:
    For most generic endpoints, the Accept: should be used to decide what content type to return. 
    If the Accept header is not present, the default will be application/json;version=1.0, 
    unless the endpoint is specifically for a binary type, in which case the default will be the binary type.
    Example:
        GET /api/v1/health HTTP/1.1
        Host: instance.example.coom
        Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8

        HTTP/1.1 200 OK
        Content-Type: text/html
        <html>
            <p>
                Instance is OK
            </p>
        </html>
    
    versus
        GET /api/v1/health HTTP/1.1
        Host: instance.example.coom

        HTTP/1.1 200 OK
        Content-Type: application/json
        {
            "status": "ok"
        }


    


types:
    base64 - string representing octets encoded in [urlsafe-base64](https://datatracker.ietf.org/doc/html/rfc4648#section-5)

mime:
    application/
        vnd.thimble.
            identity-keychain+octet-stream - a serialized IdentityKeychain
            identity-certificate+octet-stream - a serialized IdentityCertificate
            key+octet-stream - a serialized IdentityKey
            message+octet-stream - a serialized Message

/api/v1
    /health
        GET
            get the health of the instance
        :RESPONSES
            200 OK

    /instance
        /keys
            GET
                get the instance's IKR
            :RESPONSES
                200 OK
        /challenge
            POST
                request the instance to verify it's identity
            :REQ-HEADERS
                Content-Type: application/vnd.thimble.challenge+octet-stream
            :RESPONSES
                200 OK
                411 Length Required
                413 Content Too Large
                - The instance may reject challenges requesting 
                
    /user/<id:base64>
        parameters:
            id - encoded key fingerprint. 
                A user can be referred to by any key in their IKR in the instance.


        /keys
            GET
                returns the user's identity keychain
            :REQ-HEADERS
                Accept: <content-type>
                - possible values:
                    - application/json;version=<version>
                    - application/vnd.thimble.identitykeychain+octet-stream;version=<version>
                - if the version field is omitted, the latest version will be returned
                If-Modified-Since: <timestamp>
                -   reply with 304 if no changes to the keychain have been made since the given timestamp
            :RESPONSES
                200 OK
                304 Not Modified
                404 Not Found

        /activity
            GET
                get the current activity of the user

        /private-key-backup
            idk

        /report
                
    /me
        /register
            POST
                register a new identity with the instance
            :REQ-HEADERS
                Content-Type: application/vnd.thimble.identitykeychain+octet-stream
                - The keychain should have the following fields set:
                    - instanceIdentityFingerprint
                    - clientIdentityFingerprint
                    - clientIdentitySignature
                    - clientIdentity
                    - homeserverIdentity
                    - at least one key in the keychain, with the
                        first key signed by the client's identity key
            :RESPONSES
                201 Created
                409 Conflict
                    - The clientIdentityFingerprint is already registered with the instance
                422 Unprocessable Entity
                    - The keychain was unable to be verified, or the keychain was missing required fields
            :RES-HEADERS
                Content-Type: application/vnd.thimble.identitykeychain+octet-stream
                    - The response body will contain the finished IKR with the instance putting it's signature on the tbskeychainIdentifer.


        /keys
            POST
                add a new key
            :REQ-HEADERS
                Content-Type: application/vnd.thimble.key+octet-stream
            :RESPONSES
                201 Created
                409 Conflict
                    - The key is already registered with the instance
                422 Unprocessable Entity
                    - The key was unable to be verified, or the key was missing required fields
        /activity
            POST
                update the current activity of the user

    /room/<room_id:base64>
        parameters:
            id - random integer identifying a room on the instance.

        /messages
            POST
                send a new message to the room

        /messages?after=<message_id:base64>&before=<message_id:base64>
            required: after and/or before

            GET
                get a list of messages within a timeframe
            :REQ-HEADERS
                see 

        /messages/<message_id:base64>
            GET
                get the contents of a message
            :REQ-HEADERS
                Accept: <content-type>
                - possible values:
                    - application/json;version=<version>
                    - application/vnd.thimble.message+octet-stream;version=<version>
                - if the version field is omitted, the latest version will be returned
                If-Modified-Since: <timestamp>
                - reply with 304 if the message hasn't been edited
                
            :RESPONSES
                200 OK
                304 Not Modified
                404 Not Found

            :RES-HEADERS
                X-Content-Type-Options: nosniff

            PATCH
                edit a message
                    message edit history?????
        
        /blocks
            ...

        /recipients
            GET
                get a list of recipients in the room

        /events
            GET
                subscribe to a websocket of room events
                - message send events
                - message edit events
                - metadata edit events
                - user started new message block
                - user activity
                - etc.

            NOTE: although, it might make more sense to have instances automatically send these events to the homeservers of the room members for e.g. notifications, in addition to supporting the "polling" (wrong term but) style

    /auth
        /challenge
            POST
                request a challenge from the instance to verify the identity of the user
                The client sends a IdentityCertificate that uses their currently active key
            :REQ-HEADERS
                Content-Type: application/vnd.thimble.challenge+octet-stream
                Accept: application/vnd.thimble.challenge+octet-stream
            :RESPONSES
                200 OK
        
        /verify[?token-options=<token-options>]
            POST
                verify the challenge response from the user
                The client sends a IdentityCertificate that uses their currently active key
            :REQ-HEADERS
                Content-Type: application/vnd.thimble.challenge+octet-stream
                Accept: application/vnd.thimble.authtoken+octet-stream
            :RESPONSES
                200 OK
                403 Forbidden
                    - The challenge response was invalid, expired, or the token-options are not authorized
                429 Too Many Requests
                    - The user has failed to verify too many times in a short period of time

                