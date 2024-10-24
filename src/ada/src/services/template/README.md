OpenUxAS SPARK Service Design Template - README
===============================================

This README describes how to begin the design process for creating an OpenUxAS service in Ada and SPARK. In brief, the process starts with running a script to generate partially-complete source files. These files comprise the "design pattern" for any OpenUxAS service, with "TODO" comments within the source files for the service-specific implementation details. You then edit these files, filling in the TODO sections, resulting in a compilable implementation.

# Setup

You should first make sure you have a suitable development environment set up for Ada and SPARK.
To do so, run this command:

    OpenUxAS$ ./anod build uxas-ada

This command will ensure that you have GNAT on your path. It will install GNAT and SPARK, using Alire, if needed.

Each time you open a new shell in which you want to do development, you should run:

    OpenUxAS$ eval "$( ./anod printenv uxas-ada --build-env )"

This command will pull the Ada development environment into your shell, allowing you to run `gprbuild` and `gnatprove` as expected.

Finally, you should also install VS Code and the Ada extension; this repository is configured to support Ada development using VS Code.

# Instantiating the Design Template

Conceptually, the design of any OpenUxAS service consists of two parts:

1. the part that interfaces with the rest of OpenUxAS.
2. the part that implements the service functionality.

In OpenUxAS - in both C++ and Ada - the first part inherits from ServiceBase; this is the means by which it interfaces with the rest of OpenUxAS, via ZeroMQ and using LMCP.

Ideally, the second part should have no knowledge of the first: this separation of concerns allows either the service functionality or the interfacing code to change independently of the other.
In the C++ code, these two parts are intermixed.
In the Ada code, we have separated them into distinct parts.

In addition, the design for a SPARK/Ada implementation includes a third part:

3. the part that passes data between the interface and the functionality (parts one and two), ensuring that the data is SPARK-compatible.

Each of the three parts is implemented by a distinct package in a SPARK/Ada implementation. More packages could, of course, be added by the developer, depending on the specific implementation, but the three core packages will always be included.

By default with GNAT, a package declaration and corresponding body occur in two separate source files. Hence there are at least six files for any given SPARK/Ada service implementation. 

Therefore, we provide a script that will create these three packages in their six source files automatically, with package and type names reflecting the service involved and the purposes of the specific packages. 

The unit names for these generated packages are based on the service name itself. The three names are as follows, where "<Service_Name>" is a placeholder for the service name:

1. <Service_Name>_Interfacing for the interfacing part.
2. <Service_Name> for the functionality part.
3. <Service_Name>_Mailboxes for the data-passing part.

As shown, the unit name for a part two package is simply the service name itself. The name thus reflects the fact that this package implements the service functionality.

The unit name for a part three package is the service name with "_Mailboxes" appended. This package provides a data passing facility between the other two parts, using a type named "Mailbox", hence the name of the package reflects that type name.

The unit name for a part one package is the service name with "_Interfacing" appended, because this package provides the interfaces with the rest of OpenUxAS. However, that is not the full name for the package. The package for the first part is a hierarchical child package, a "child" of a package named "UxAS.Comms.LMCP_Net_Client.Service". Hence the complete package name is 
"UxAS.Comms.LMCP_Net_Client.Service.<Service_Name>_Interfacing".

For example, if the service name is Route_Aggregator, the package unit names will be as follows:

1. UxAS.Comms.LMCP_Net_Client.Service.Route_Aggregator_Interfacing
2. Route_Aggregator
3. Route_Aggregator_Mailboxes

The names of the source files must reflect the unit names contained within them. That is the default naming scheme in GNAT. Other naming schemes are supported, but the script generates names conforming to the default scheme.

Therefore, the source file names for the three packages will reflect the service name, with any dots in the unit names changed to dashes (for part one packages). The package declarations and bodies are in separate files, with file extensions of ".ads" and ".adb" respectively. 

Continuing the Route_Aggregator service name example, the six generated files will be named as follows:

1. uxas-comms-lmcp_net_client-service-route_aggregator_interfacing.ad[s|b]
2. route_aggregator.ad[s|b]
3. route_aggregator_mailboxes.ad[s|b]

The script is named `instantiate`.

When invoking the script you must specify the service name. The name must be a single, syntactically legal Ada identifier. It is not a string and is not specified to the script within quotes. Furthermore, although Ada is not case-sensitive, the specified name should be given in mixed case format, as, for example, we have done with Route_Aggregator. This casing is not required but the generated code uses it throughout, so for the sake of readability we suggest you use that format as well.

For example, for the route aggregator service you would specify Route_Aggregator as the single argument to the script (i.e., without any  quotes). Note the underscore.

    OpenUxAS/src/ada/src/services$ template/instantiate Route_Aggregator

That will generate the six SPARK/Ada source files.

The script uses the file `template.txt`, contained in this directory, to guide the generation.
Users may alter this file to change the generated design pattern source files, although the need to do so is not expected.

As a result, assuming the invocation illustrated above, the script will generate these files in a new subdirectory `route_aggregator` under `OpenUxAS/src/ada/src/services`:

    route_aggregator.adb
    route_aggregator.ads
    route_aggregator_mailboxes.adb
    route_aggregator_mailboxes.ads
    uxas-comms-lmcp_net_client-service-route_aggregator_interfacing.adb
    uxas-comms-lmcp_net_client-service-route_aggregator_interfacing.ads

There is one final name to mention: the ServiceName. This is the name of the service as expected in the rest of OpenUxAS (in C++) and as it appears in the OpenUxAS XML configuration file. The name appears in the generated code as a string literal in the interfacing package, specifically as the value of the `Type_Name` constant.

The script generates the value for the constant, automatically. In particular, the script first removes all the underscores in the name passed as the script argument, and then appends "Service" to the result.

For example, with the following invocation:

    OpenUxAS/src/ada/src/services$ template/instantiate Route_Aggregator

the generated service name literal would be "RouteAggregatorService":

   Type_Name : constant String := "RouteAggregatorService";

One of the "TODO" entries in the generated source files is to verify that the string literal exactly matches the value expected by the rest of OpenUxAS, in particular in the OpenUxAS XML configuration file.

Because the generated files are incomplete, they are not immediately compilable. Users must fill in the missing parts by editing the files. The "TODO" entries facilitate this effort.

In the sections that follow, we describe how to modify the generated files and make other changes necessary to include them in the Ada executable.
We will refer to the names of the files, and the names of the packages within those files, using the <Service_Name> placeholder.
For example, we will refer to the package that provides the service functionality (part 2) as "package <Service_Name>", and corresponding files as "<service_name>.ads" and "<service_name>.adb" for the spec and body respectively.
Likewise, we will refer to "package <Service_Name>_Mailboxes" and "package <Service_Name>_Interfacing" (although the actual, full package name would be "UxAS.Comms.LMCP_Net_Client.Service.<Service_Name>_Interfacing").

## Add Messages

If your implementation requires SPARK-compatible LMCP messages that have not previously been used in any existing SPARK/Ada UxAS service, you must manually add support for them. To do this, you must:

1.  Add a type definition for the message to the SPARK package `LMCP_Messages` in
    `/src/services/spark/lmcp_messages.ads`. Note that you will need to
    recursively add type definitions for any field of the message whose type
    is a also message (or message array) that is currently undefined.

2.  Add a function to convert the corresponding Ada LMCP message to the newly
    defined SPARK-compatible one to the specification and body of package
    `LMCP_Message_Conversions` in
    `/src/services/spark/lmcp_message_conversions.{ads,adb}`. Note that you
    will need to recursively add functions for any fields of the message for
    which such conversion functions are not currently defined. By convention,
    the name of such functions should be `As_<MessageType>_Message`. See
    existing functions in this package for examples.

You can publish SPARK-compatible LMCP messages directly from SPARK portions of a service implemented in SPARK package `<Service_Name>` using the `sendBroadcastMessage` procedure from that same package.
This procedure relies on the `As_Object_Any` function from package `LMCP_Message_Conversions`, which in turn relies on manually defined functions for converting SPARK-compatible LMCP messages back to Ada LMCP messages.
If you want to be able to broadcast SPARK-compatible LMCP messages directly from SPARK portions of your service, you will therefore need to:

1.  Add a function to the body of package `LMCP_Message_Conversions` to
    convert the SPARK-compatible LMCP message type back to an access type or
    class-wide access type for the original Ada LMCP message type.
    -   By convention, such functions are named `As_<MessageType>_Any` or
        `As_<MessageType>_Acc` depending on whether they return a classwide
        access type or simply an access type.
    -   Unless it is needed external to the package (which is unlikely), you
        should put the function declaration in the package body, not the
        package specification. See package `LMCP_Message_Conversions` for
        examples.
    -   You will need to recursively add functions for any fields whose types
        are messages for which such conversion functions are not currently
        defined.

2.  Using the previous function, add a case for the message type to
    `function As_Object_Any` in the body of `LMCP_Message_Conversions`.
    Recursively add cases for any fields whose types are messages that
    are not currently handled in this function.

## Modify Generated Files

The following instructions walk through how to modify the contents of each of the generated service template files. Note that each instruction has a corresponding `__TODO__` comment in the template file itself, along with examples in a corresponding `__Example__` comment.

### Communication

Package `<Service_Name>_Mailboxes` declares a "mailbox" type for the service and associated procedures for using it, including a procedure `sendBroadcastMessage` for sending Ada LMCP messages over ZeroMQ.
The content is boilerplate, apart from occurrences of the service name, so the two generated source files are complete and ready for use.

### Ada

Package `UxAS.Comms.LMCP_Net_Client.Service.<Service_Name>_Interfacing`
implements the Ada portion of the service. This portion of the service does
the following:

-   Declares and initializes the service's main tagged type
    `<Service_Name>_Service`, which inherits from `Service_Base` and includes
    the service's mailbox, along with SPARK-compatible state and configuration
    data.
-   Declares and initializes the service's mailbox, state, and configuration.
    The mailbox is initialized through the `Initialize` procedure, and the
    state and configuration is initialized mainly through default initial
    values and the `Configure` procedure.
-   Declares and defines the name by which the service is referred to in the
    OpenUxAS configuration file (via the constant `Type_Name`).
-   Declares and defines the service's main LMCP message processing loop,
    making use of a mix of local Ada message handlers and local helper
    subprograms, along with SPARK message handlers and SPARK helper subprograms
    from package `<Service_Name>`.

The steps to implement this portion of the service from the template files
are as follows:

1.  Within `uxas-comms-lmcp_net_client-service-<service_name>_interfacing.ads`:
    1.  Add `with` clauses for any additional packages needed in this package's
        specification to the top of the file.
    2.  Verify the name of the service in line that reads
        `Type_Name : constant String := "<ServiceName>Service"`.
        This is the name the Service Manager will use
        to load the service, and it is therefore the name by which you will
        refer to the service in the SPARK/Ada OpenUxAS configuration file.
    3.  Add any additional service-specific fields to record
        `<Service_Name>_Service`, which already has fields `Config`, `Mailbox`,
        and `State`. Note that additional fields may not be needed.
2.  Within `uxas-comms-lmcp_net_client-service-<service_name>_interfacing.adb`:
    1.  Add `with` clauses for any additional packages needed only within this
        package's body. This is likely to include LMCP messages, e.g. from package
        AFRL.CMASI, and it may include other packages as well.
    2.  Declare any subprograms used locally within the package. This should
        include procedures for handling Ada LMCP messages, which by convention
        should be named `Handle_<MessageType>_Msg`. Also by convention, their
        definitions are deferred until closer to the end of the file.
    3.  In `procedure Configure`,
        1.  Add any necessary service-specific configuration logic. This is
            likely to include logic to set service-specific configuration
            parameters read from the OpenUxAS XML configuration file.
        2.  Subscribe to any messages this service should receive.
    4.  Define any Ada LMCP message handling procedures that were declared
        earlier in the package.
        -   In some cases, these may essentially be wrappers around SPARK LMCP
            message handling procedures from package `<Service_Name>`. In such
            cases, the Ada LMCP message handler should use the
            `As_<MessageType>_Message` function from package
            `LMCP_Message_Conversions` to convert the Ada LMCP message to a
            SPARK-compatible LMCP message before calling the SPARK LMCP message
            handler, which by convention should be named `Handle_<MessageType>`.
        -   In some cases, there may be no Ada LMCP message handler for a
            particular type of message that the service subscribes to. Instead,
            there is only a handler for the analogous SPARK LMCP message, which
            is called by the `Process_Received_LMCP_Message` procedure described
            in the next step.
    5.  Within procedure `Process_Received_LMCP_Message`:
        1.  Add an `if-elsif` block to handle every type of message this service
            subscribes to. For each type of message, call either a local Ada
            LMCP message handler or a SPARK LMCP message handler from package
            `<Service_Name>`.
        2.  Add any additional processing logic that should occur every time
            after a message is received.
    6.  Define any other local procedures that were declared earlier in the
        package.


### SPARK

SPARK package `<Service_Name>` contains SPARK subprograms and associated data
structures used to implement key behaviors of the service that can be
formally verified. Public subprograms that update the service's state tend to
be called by the Ada portion of the service in package
`UxAS.Comms.LMCP_Net_Client.Service.<Service_Name>_Interfacing` when a new
message is received or other events occur, e.g. a timer within the service
triggers. The `<Service_Name>` SPARK package likely includes a significant
amount of ghost code for specification and to help guide proof, and it may
also include helper subprograms that decompose the processing and make proof
more tractable. The steps to implement this portion of the service from the
template files are as follows:

1.  Within `<service_name>.ads`:
    1.  Add `with` clauses for any additional packages needed in this package's
        specification to the top of the file.
    2.  Define the types needed for this SPARK package.
    3.  Declare the fields of record `<Service_Name>_Configuration_Data`, which
        holds configuration information for the service. This configuration
        information is generally from the OpenUxAS XML configuration file and
        is initialized by the Ada portion of the service using the `Configure`
        procedure.
    4.  Declare and optionally provide default values for the fields of record
        `<Service_Name>_State`, which holds state information for the
        service. This information tends to change as messages are processed and
        computations are performed.
    5.  Declare public subprograms needed in this package. These tend to
        include (a) procedures to handle SPARK-compatible LMCP messages; (b)
        subprograms that implement major behaviors of the service that need to
        be accessible by Ada portions of the service; and (c) ghost code for
        specification, especially for pre- and postconditions of subprograms
        for (a) and (b).
2.  Within `<service_name>.adb`:
    1.  Add `with` clauses for any additional packages needed in this package's
        body to the top of the file.
    2.  Add any local types or `use` clauses you would like to have.
    3.  Declare and define bodes for any local subprograms that are only used
        within the body of this package. This may include helper subprograms or
        ghost code (e.g. lemmas) to help with proof.
    4.  Define bodies for any subprograms that were declared in the package
        specification. These are likely to include procedures to handle
        SPARK-compatible LMCP messages (by convention named
        `Handle_<MessageType>`), along with other SPARK subprograms needed by
        the service.
        -   Note that procedures that send SPARK-compatible LMCP messages
            directly should include the service's mailbox as a parameter.
        -   Also, as a general tip for proof, subprograms that have complex
            contracts and operate on the state should in their implementations
            rely on helper subprograms that operate over **only** the required
            fields of the state and have contracts that can be leveraged for
            proof of the original subprogram's contract. This modularizes proof
            and minimizes context for the provers, making proof more tractable.

# Adding the Service to Main

1.  Include your service in `src/ada/src/main/uxas_ada.adb` by adding
    the following lines to the file, after analogous lines for other
    services:

        with UxAS.Comms.LMCP_Net_Client.Service.<Service_Name>_Interfacing;
        pragma Unreferenced (UxAS.Comms.LMCP_Net_Client.Service.<Service_Name>_Interfacing);

The with-clause is necessary for the package to be part of the Ada main's executable image.
The pragma informs the compiler that it need not generate a warning about the fact that the package is not otherwise referenced with the body of the main procedure.

# Build and Run SPARK/Ada OpenUxAS

1.  Once you're ready to start using your code in conjunction with other
    anod-managed code, change to the main OpenUxAS directory, and build it with
    command: `./anod build uxas-ada`

2.  You can demonstrate your code with OpenAMASE by setting up an example in
    the `examples` directory that can be run from the main OpenUxAS directory
    using the command: `./run-example`
    -   See `examples/02a_Ada_WaterwaySearch` for an example that uses SPARK/Ada
        OpenUxAS alongside C++ OpenUxAS and OpenAMASE to do a line search. Files
        that affect running of the example include the configuration files for
        both the SPARK/Ada and C++ versions of OpenUxAS (`cfg_ada.xml`,
        `cfg_cpp.xml`), the configuration file for OpenAMASE
        (`Scenario_WaterwaySearch.xml`), the configuration file for `run-example`
        (`config.yaml`), and messages in the `MessagesToSend` directory, which
        are injected based on the configuration of the `SendMessagesService` in
        the OpenUxAS configuration file(s).

# Adding Proofs to CI

Once you've completed the development of your SPARK service and have proved all of your properties, you can add your new proofs so that they will be run by `run-proofs`.
This will also ensure that your proofs are run as part of continuous integration.

Create a new directory under `tests/proof/proofs/`, like this:

    OpenUxAS$ mkdir tests/proof/proofs/service_name

Then create two files: `test.yaml` and `test.out`.

In `test.yaml`, you will:

* name the spec files associated with the SPARK part of your service,
* indicate the proof level
* indicate a timeout, if any

Like this:

    filenames: ["service_name.ads"]
    level: 2

The options you pick should match those you used when you completed the proofs.
See the existing entries under `tests/proof/proofs/` for other examples.

In the `test.out`, you may list expected proof failures.
See the existing entries under `tests/proof/proofs/` for examples.

Now, when you run `run-proofs`, you will see the report of the proofs of your service.
