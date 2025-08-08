Xnamespace extension v1.0
=========================

This extension separates clients into several namespaces which are isolated from each other.
It is similar to Linux's kernel namespaces.

Namespaces have their own selections, and clients cannot directly interact
(send messages) or access another client's resources across namespace borders.
The only exceptions are clients in the root namespace.

# Configuration

Namespaces are defined in a separate configuration file, which is loaded at
server startup.
There is no dynamic provisioning in this version yet.
The extension is enabled when a namespace config is passed to the Xserver via the
`-namespace <fn>` flag.

See `Xext/namespace/ns.conf.example` for a configuration file example.

# Authentication / Namespace assignment

Assignment of clients into namespaces is done by the authentication token the
client is using to authenticate; so, token authentication needs to be enabled.


# How it works

**XNamespace (XN)** uses the **X Access Control Extension Specification (XACE)** to hook into the X server's functions. 
Whenever a client tries to access an X server resource, the client's namespace is checked for the correct privileges. 
If the client is in the correct namespace with the appropriate permissions, access to the resource is granted; 
otherwise, XN will deny access to that resource.

## XACE Callbacks Enums used by XN
- XACE_CLIENT_ACCESS (XCA)
- XACE_DEVICE_ACCESS (XDA)
- XACE_EXT_DISPATCH (XED)
- XACE_EXT_ACCESS (XEA)
- XACE_PROPERTY_ACCESS (XPA)
- XACE_RECEIVE_ACCESS (XRecA)
- XACE_RESOURCE_ACCESS (XResA)
- XACE_SEND_ACCESS (XSenA)
- XACE_SERVER_ACCESS (XSerA)


## Consequences of Unallowed access

| Property          | XCA | XDA | XED                                            | XEA                        | XPA | XRecA                                           | XResA                     | XSenA | XSerA |
|-------------------|-----|-----|------------------------------------------------|----------------------------|-----|-------------------------------------------------|---------------------------|-------|-------|
| allowMouseMotion  | N/A | N/A | N/A                                            | N/A                        | N/A | Status is set to BadAccess and client is logged | N/A                       | N/A   | N/A   |
| allowShape        | N/A | N/A | Status is not changed and the client is logged | Status is set to BadAccess | N/A | N/A                                             | N/A                       | N/A   | N/A   |
| allowTransparency | N/A | N/A | N/A                                            | N/A                        | N/A | N/A                                             | Background will be opaque | N/A   | N/A   |
| allowXInput       | N/A | N/A | Status is not changed and the client is logged | Status is set to BadAccess | N/A | N/A                                             | N/A                       | N/A   | N/A   |
| allowXKeyboard    | N/A | N/A | Status is not changed and the client is logged | N/A                        | N/A | N/A                                             | N/A                       | N/A   | N/A   |

## Examples
### Permissions given by example file
Using the ns.conf.example file.

| Container | mouse-motion | shape | xinput |
|-----------|--------------|-------|--------|
| root      | ✔️           | ✔️    | ✔️     |
| xeyes     | ✔️           | ✔️    | ✔️     |
| xclock    |              |       |        |

### Example of Clients trying to access xinput and each other from different namespaces

| Client  | Namespace | Access to xinput    | Communication with others          |
|---------|-----------|---------------------|------------------------------------|
| Client1 | root      | Succeeds (implicit) | Can communicate with App2 and App3 |
| Client2 | xeyes     | Succeeds (explicit) | Cannot access App1 or App3         |
| Client3 | xclock    | Fails               | Cannot access App1 or App2         |