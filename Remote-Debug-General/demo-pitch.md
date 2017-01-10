Cross-platform Remote Debugging
===============================

For a five minute video introduction to remotely debugging Python code, see this video.

[![Deep Dive: Cross-Platform Remote Debugging](VideoThumbnails/RemoteDebugging.png)](https://youtu.be/y1Qq7BrV6Cc)

Popular Data Science Scenario
-----------------------------
Often data scientists and ML engineers execute a code on a remote VM to take advantage of greater compute power required by techniques like Deep Learning. For instance, they may have a code running TensorFlow or the Cognitive Toolkit remotely only sending the final outcomes back to a client. With remote debug features, developers can debug such code just like they can debug code running locally without making changes to their workflow or experiment set-up. 

Feature Overview
--------

Visual Studio can launch and debug Python applications locally, and can attach to already-running CPython processes on the local machine (or a remote Windows machine using the [Visual Studio Remote Debugging Monitor](http://msdn.microsoft.com/en-us/library/xf8k2h6a.aspx)). It is also possible to attach to code running on a different operating system, device, or a Python implementation other than CPython by using the [ptvsd](https://pypi.python.org/pypi/ptvsd) library.

In this mode, the Python script being debugged also hosts the debug server to which the IDE can attach. It requires a small modification to the source code of your script (to import and enable the server), and may require network or filewall configuration to allow a TCP connection to be made between the IDE and the debuggee.

Demo Overview
----------------------------------
1. Prepare the script for debugging
2. Attach remotely from Python Tools 
3. Secure the debugger connection (optional, but good to highlight to enterprise customers)

Preparing the script for debugging
----------------------------------
The accompanying Python script will be used for demonstration purposes (taken without modifications from the Python wiki):
 
The Python Tools debug server is contained in the `ptvsd` package that comes with Python Tools (search `ptvsd` in Start to find the folder) or can be installed from [PyPI](https://pypi.python.org/pypi/ptvsd) using `pip install ptvsd`.

After the package is made available for import on the remote machine, add the following two lines of code to enable remote debugging in at the start of your script:

```python
import ptvsd
ptvsd.enable_attach(secret='my_secret')
ptvsd.wait_for_attach
```

The `secret` parameter passed to `enable_attach` is used to restrict access to the running script. When attaching, you will have to specify it in Visual Studio or the connection will be denied. To disable this and allow anyone to connect, use `enable_attach(secret=None)`.

Attaching remotely from Python Tools
------------------------------------

The most common scenario for remote debugging is to set a breakpoint in the source and run the script until that breakpoint is hit. 

IMPORTANT: A local copy of the source file being debugged on the machine running Python Tools. It does not matter where the file is located, but its name should match the name of the actual script on the remote machine that will be attached to: 

![Setting a breakpoint](Images/RemoteDebuggingBreakpointSet.png)

Attaching is done using the Debug -> Attach to Process command, which opens the "Attach to Process" dialog. From the "Transport" combo box select "Python remote debugging": 

![Choosing the remote debugging transport](Images/RemoteDebuggingTransport.png)

After that, the "Qualifier" textbox and the "Available Processes" list will be blank, and "Transport information" will provide a brief reminder of the steps necessary to set up remote debugging. Type the address of the remote machine, prefixed with the secret that was defined in the script, into the "Qualifier" textbox, and press Enter:

![Entering the qualifier](Images/RemoteDebuggingQualifier.png)

Find the remote Python process in the "Available Processes" list. An error at this stage typically indicates that the secret did not match, the `ptvsd` version does not match that being used by PTVS, or a connection could not be established. 

Common Failure Case: Remote machine has a firewall that is blocking the debug server port (default is 5678) open. Reconfigure the firewall or use a different port; the latter can be done by explicitly specifying it in the call to `enable_attach` in the `address` parameter, e.g.:

```python
ptvsd.enable_attach(secret = 'my_secret', address = ('0.0.0.0', 8080))
```

The address format is the same as the one used by the standard Python module socket for sockets of type `AF_INET`; see its [documentation](http://docs.python.org/3/library/socket.html#socket-families) for details. 

Once the process appears in the list, double-click to attach to it. For the example script shown above, entering a number will cause the breakpoint to be hit:

![Breakpoint is hit](Images/RemoteDebuggingBreakpointHit.png)

From there on, you can use all the usual debugging features offered by Python Tools. 

If you stop debugging, Visual Studio detaches from your script, but the script will continue running on the remote machine.

Securing the debugger connection with SSL
-----------------------------------------

Obtain an SSL certificate to generate a self-signed certificate, as [described](http://docs.python.org/3/library/ssl.html#self-signed-certificates) in the documentation for Python standard module `ssl`. 

Note: To prevent MITM attacks, such a generated certificate will also have to be added to the CA root store on the Windows machine running Python Tools. This can be done using the Certificate Manager (certmgr.msc) as described [here](http://windows.microsoft.com/en-us/windows/import-export-certificates-private-keys). Note that you will need to have a separate certificate file (not combined with the private key in a single file) to import. 

Update the call to `enable_attach` in your script by means of `certfile` and `keyfile` parameters, which have the same meaning as for the standard Python function `ssl.wrap_socket`. For example, if the certificate file is called `my_cert.cer`, and the key file is called `my_cert.key`, use: 

```python
ptvsd.enable_attach(secret='my_secret', certfile='my_cert.cer', keyfile='my_cert.key')
```

The attach process is exactly the same as described earlier, except that, instead of using the `tcp://` scheme in the Qualifier, use `tcps://`: 

![Choosing the remote debugging transport with SSL](Images/RemoteDebuggingQualifierSSL.png)

If you did not add the certificate to the CA root store, you will get a warning message: 

![SSL certificate warning](Images/RemoteDebuggingSSLWarning.png)

You may choose to ignore this and proceed with debugging – the channel will still be encrypted against eavesdropping, but ignoring the warning opens you to a possibility of a MITM attack.
