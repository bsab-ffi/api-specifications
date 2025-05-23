# Handling Connectivity Check

By following this section of the General FFI API Conventions, we ensure that all participants in the FFI network enables the verification of the connectivity for a newly onboarded FFI-Requestor client in production. Based on a common standardized json based FFI connectivity-check message.

## General Principles
There is a need to be able to verify the connectivity for an onboarded FFI-client , representing either a bank or a governmental agency, in production without having to send a real message that might be lost or delayed if the onboarding in some aspect was not successful.
A simple and robust method to verify the connectivity for an onboarded FFI-client is to have a common standardized json based connectivity-check message.
This message-type should all implementations of the FFI-api support for all FFI-supporting banks, to use as a means of verifying onboarding of a new part in production, without jeopardizing any real live data.

## ConnectivityCheck UseCases/Scenarios and its Benefits:
- Verify a newly onboarded FFI-client that authentication and authorization is working incl. the certificate, mTLS, oauth, message and version authorization etc
- May  be used for internally trigger a connectivity check to see that the FFI-server orchestrator is up and working
- May be used for external FFI parts to verify the FFI-server is alive i.e. up and working for a seleected FII-server
- When a new version of the FFI-core API is implemented, it is possible to test that both the new and current APIs is working as intended
- Verify full connectivity after a certificate renewal has been done

## FFI-Server Responsibilities
Implement the ConnectivityCheck message-type and corresponding json-schema in the FFI-server, so that the FFI-server can receive and respond to ConnectivityCheck messages from FFI-clients.

## FFI-Client Responsibilities
Implement the ConnectivityCheck message-type and corresponding json-schema in the FFI-client, so that the FFI-client can send ConnectivityCheck messages to the FFI-server and receive and acknowledge ConnectivityCheck responses as part of the verification of the onboarding process.

## ConnectivityCheck Step-by-Step
1) The newly onboarded FFI-client POST a ConnectivityCheck message with message-version to the `/requests` endpoint
2) The FFI-server will verify the ConnectivityCheck message with corresponding json-schema and will then make a normal response with a requestId etc
3) Instead of sending the message internally to the processing services within the bank, a ConnectivityCheck response message is automatically created and saved
4) The FFI-client will call the `/responses` endpoint and there info about the ConnectivityCheck response message will be returned including the responseId
5) The ResponseId is used to fetch the automatically created ConnectivityCheck response message 
6) The FFI-client will verify the response with json schema
7) The FFI-client will POST an acknowledgement
8) The FFI-server will delete the generated ConnectivityCheck response message  
9) Celebration is in place, since the connectivity for the newly onboarded FFI-client was successful! :-)  

## ConnectivityCheck Message-Type
Schema access to the **ConnectivityCheck** message-type including examples and corresonding json-schemas is provided through the Schema Registry

**Message-type name:** ConnectivityCheck

## Federation-Specific Parameters
This convention does not have any federation-specific parameters.
