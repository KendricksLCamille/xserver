Xnamespace extension v1.0
=========================

This extension separates clients into several namespaces which are isolated from each other.
It is similar to Linux's kernel namespaces.

Namespaces have their own selections, and clients cannot directly interact
(send messages) or access another client's resources across namespace borders.
The only exceptions are clients in the root namespace.

Configuration
-------------

Namespaces are defined in a separate configuration file, which is loaded at
server startup.
There is no dynamic provisioning in this version yet.
The extension is enabled when a namespace config is passed to the Xserver via the
`-namespace <fn>` flag.


See `Xext/namespace/ns.conf.example` for a configuration file example.

Authentication / Namespace assignment
-------------------------------------

Assignment of clients into namespaces is done by the authentication token the
client is using to authenticate; so, token authentication needs to be enabled.


How it works
---------------

**XNamespace (XN)** uses the **X Access Control Extension Specification (XACE)** to hook into XLibre's functions.
So everytime an XLibre resource is used, if XN is enabled, it will check if that application is allowed access to a
resource.
If it is in the correct namespace with the correct permissions, the resource is available else XN will 
deny access to that resource.

Example
------------
Using the ns.conf.example file.

| Container | mouse-motion | shape | xinput |
|-----------|--------------|-------|--------|
| root      | ✔️           | ✔️    | ✔️     |
| xeyes     | ✔️           | ✔️    | ✔️     |
| xclock    |              |       |        |



| Application | Namespace     | Access to xinput | Communication with others                     |
|--------------|---------------|------------------|----------------------------------------------|
| App1         | root          | Succeeds (implicit) | Can communicate with App2 and App3       |
| App2         | xeyes         | Succeeds (explicit) | Cannot access App1 or App3               |
| App3         | xclock        | Fails             | Cannot access App1 or App2               |