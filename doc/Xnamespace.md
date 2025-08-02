Xnamespace extension v1.0
=========================

This extension separates clients into several namespaces (a bit similar to
Linux's kernel namespaces), which are isolated from each other. 

For example, namespaces have their own selections, and clients cannot directly interact
(send messages) or access another client's resources across namespace borders.

An exception is the `root` namespace, which completely is unrestricted.

Configuration
-------------

Namespaces are defined in a separate configuration file, which is loaded at
server startup (no dynamic provisioning in this version yet).
The extension is enabled when a namespace config is passed to the Xserver via the
`-namespace <fn>` flag.


See `Xext/namespace/ns.conf.example` for a configuration file example.

Authentication / Namespace assignment
-------------------------------------

Assignment of clients into namespaces is done by the authentication token the
client is using to authenticate.
Thus, token authentication needs to be enabled.


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


If a user has three applications
- App1 is in the **root** namespace
- App2 is in the **xeyes** namespace
- App3 is in the **xclock** namespace

If all the application attempts to access a resource or function related to xinput,
- App1 will succeed because it is implicitly allowed in the **root** namespace.
- App2 will succeed because it is explicitly allowed in the **xeyes** namespace.
- App3 will fail because it isn't allowed in that namespace. 

If all the apps attempt to communicate with each other, the below will occur.

App1 will be able to communicate with App2 and App3.
App2 and App3 will not be able to access App1 nor each other due to being in lower and different namespaces.
