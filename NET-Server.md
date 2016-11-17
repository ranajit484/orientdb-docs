---
search:
   keywords: ['.NET', 'C#', 'C Sharp', 'OServer']
---

# OrientDB-NET - `OServer`

In order to interact with OrientDB from within your C#/.NET application, you need to create an instance of the `OServer` class.  This provides you with an interface to use when operating on the OrientDB Server.


## Initializing OServer

With OrientDB-NET, the server interface is controled through the `OServer` class, which can be found in the `Innov8tive.API` library.

**Syntax**

```
OServer(    string <hostname>, 
            int <port>, 
            string <userName>, 
            <string userPasswd>)
```

- **`<hostname>`** Defines the host that you want to connect to, such as `localhost` or the IP address on which OrientDB is running.
- **`<port>`** Defines the port you want to connect to, such as 2424.
- **`<userName>`** Defines the Server user name.
- **`<userPasswd>`** Defines the Server user password.

**Example**

In the interest of abstraction, you might create a class with methods to handle common OrientDB Server operations.  For instance, say you want a method that creates and returns a new `OServer` instance,


```csharp
public static class Server
{
   private static string _hostname  = "localhost";
   private static int _port         = 2424;
   private static string _user      = "root";
   private static string _passwd    = "root_passwd";

   public static OServer Connect()
   {
      server = new OServer(_hostname, _port, 
            _root, _root_passwd);
      return server;
   }

}
```

With this `Server` class, you can use the `Connect()` method to retrieve a new instance of `OServer`.  You can then use this instance in various database operations.


## Using OServer

Once you have created an instance of `OServer` within your application, you can use its methods in performing various operations on databases, including,

- [**`CreateDatabase()`**](NET-Server-CreateDatabase.md)
- **`DatabaseExists()`**
- **`DropDatabase()`**
- **`ConfigGet()`**
- **`ConfigSet()`**
- **`Dictionary()`**

### Closing OServer

When you are finished using an instance server interface, you can close it to free up system resources.  The `OServer` class provides two methods to close a server connection.

- **`Close()`**
- **`Dispose()`**

Given one is an alias, you can use whichever you find most familiar.  For instance, say that you have creating an `OServer` instance in your application and set it on the `server` object.  To dispose of this instance, call one of the above methods on that object.

```csharp
// Close Server
server.Close();
```