# Introduction #

The Sylverant Patch Server is configured with an XML file, much like the rest of Sylverant. The XML file contains all of the patches available, the welcome messages, and the configuration of the server's address. The general format of the configuration file is as shown below, and usually would go in `/usr/local/share/sylverant/config/patch_config.xml`

```
<?xml version="1.0" encoding="UTF-8"?>

<!DOCTYPE patch_config PUBLIC "-//Sylverant//DTD Patch Configuration 1.0//EN"
  "http://sylverant.net/dtd/patch_config1/patch_config.dtd">

<patch_config>
    <server addr="IPv4 Address" ip6="IPv6 Address" />
    <versions pc="true" bb="true" />
    <welcome version="PC">PC Welcome Message</welcome>
    <welcome version="BB">Blue Burst Welcome Message</welcome>
    <patches version="PC" dir="PC Patch Directory">
        <!-- Simple Patch (no conditionals) -->
        <patch enabled="true">
            <clientfile>Client-Side File</clientfile>
            <file>Server-Side File</file>
            <checksum>Server-Side File Checksum</checksum>
            <size>Server-Side File Size</size>
        </patch>
        <!-- Conditionally applied patch -->
        <patch enabled="true">
            <clientfile>Client-Side File</clientfile>
            <if checksum="Client-Side Checksum">
                <file>Server-Side File</file>
                <checksum>Server-Side File Checksum</checksum>
                <size>Server-Side File Size</size>
            </if>
            <if checksum="Client-Side Checksum 2">
                <file>Server-Side File 2</file>
                <checksum>Server-Side File Checksum 2</checksum>
                <size>Server-Side File Size 2</size>
            </if>
            <!-- Optional Catch-all -->
            <else>
                <file>Server-Side File 3</file>
                <checksum>Server-Side File Checksum 3</checksum>
                <size>Server-Side File Size 3</size>
            </else>
        </patch>
    </patches>
    <patches version="BB" dir="Blue Burst Patch Directory">
        <!-- See the PC example above -->
    </patches>
</patch_config>
```

The variables of the file are described below.

  * IPv4 Address: The publicly accessible IPv4 address of the server.
  * IPv6 Address: The publicly accessible IPv6 address of the server (if any). If you don't have one of these, leave the attribute out all together.
  * PC Welcome Message: The message to show to PSOPC users when they connect to the patch server. This can be formatted in any way you want, can can have embedded newlines and such. You get about 2048 bytes worth of characters here. This should be in UTF-8.
  * Blue Burst Welcome Message: Same as above, but for PSO Blue Burst users.
  * PC Patch Directory: The directory where PSOPC patch files can be found. Usually relative to `/usr/local/share/sylverant`.
  * BB Patch Directory: Same as above, but for PSO Blue Burst.
  * Client Side File: The name of the file to check on the client. This is relative to the root of their PSO install (so usually relative to `C:\Program Files\SEGA\PhantasyStarOnline\` on PSOPC).
  * Server-Side File: The name of the file on the server. This is relative to the patch directory for whichever version the file is associated with.
  * Server-Side File Checksum: The CRC32 of the file on the server.
  * Server-Side File Size: The length (in bytes) of the file on the server.
  * Client-Side Checksum: The CRC32 of the file on the client. This is used to select from one of many patches depending on which file the user has on their system. For instance, this was used on the main Sylverant server to distribute the Ultimate-mode map fix patch while maintaining several common sets of patches for the file. Note that if the `<else>` tag is provided and none of the `<if>` tags match, that file will be sent to the user. The `<else>` is optional.

The rest of the attributes should be self-explanatory. Any tags that have `"true"` set can also have `"false"` if you so desire. Note that a patch that has `enabled="false"` set will be ignored entirely (as should be expected).