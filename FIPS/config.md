| FIP   | Title         | Status | Category               | Author                                     | Created    |
| ----- | ------------- | ------ | ---------------------- | ------------------------------------------ | ---------- |
| -     | Config  | Draft  | Core       | Who Soup \<<who.soup@gmail.com>\><br>Jay Cheroske \<<jay@bedrocksolutions.io>\>       | 20190520   |


# Summary

Unifiying and simplifying the way the config file (`factomd.conf`) and the command line parameters are loaded. Settings are split into groups, ie Core, P2P, Database, with individual groups for every network that override on a per-network basis. All settings available in the config are also command line parameters, with the command line parameter superceding the configuration setting. Command line parameters are named `-group.name` with the option of short names for select, high use settings.

# Motivation

At the moment, it's very inconvenient to add new flags to the node since parsing is spread out over several files and it's not clear where or which settings are overwritten in what way. The config file is defined in `util/config.go` and parsed by `state/state.go` during `engine/NetStart.go`, which will initialize the state. Parameters are parsed by `engine/factomParams.go` in `main()` and then overwritten throughout `NetStart`. Adding a config setting with flag requires editing three different packages and four files. 

If you are running multiple nodes on the same machine, such as for development work, it's also not possible to use the same configuration for every use-case (main vs testnet) and files have to be swapped. Being able to set network-specific settings would enable one config file for multiple networks. 


# Specification

The config file and command line parameters will be evaluated before the factomd node is run

1. Parse the command line flags and check for valid names, stop execution if there are invalid names or values
2. Create a configuration object with default values
3. Check if the "config" parameter is present, otherwise use default path, and parse the file
4. Overwrite the default configuration with values from the config
5. Determine the network from the config file and command line flag
6. Overwrite the config default values with custom values from the network's group
7. Overwrite settings with the ones specified by the command line flags
8. The configuration is passed to the Factomd() call that loads the node

The configuration file consists of the following sections and settings, which are all case insensitive except for the names of the networks:
```
; ------------------------------------------------------------------------------
; Configurations for factomd
; ------------------------------------------------------------------------------
; All settings are case insensitive and you can override specific settings in
; the factomd section with network-by-network settings by adding a 
; [factomd.NETWORKNAME] category, e.g.: [factomd.MAIN] or [factomd.fct_community_test]
; These settings will only take effect for that network
;
; All settings are case-insensitive with a command line equivalent of "--name=value",
; e.g.: "--blocktime=10m"
;
; Time-based variables allow semantic input between milliseconds (ms), seconds (s)
; minutes (m), hours (h), days (d), defaulting to seconds:
;   30ms = 30 milliseconds
;   180 = 180s = 3m
;   48h = 2d
;
[factomd]
; ---------------- GLOBAL ----------------
; The name of the network to connect to, such as MAIN, LOCAL, or fct_community_test. default = MAIN
network = MAIN

; The directory to keep factom data in. If left blank it defaults to ~/.factom/m2/ on *nix
; and %HOMEPATH%/.factom/m2/ on windows
homeDir = 

; ---------------- CONSENSUS ----------------
; The time to build one directory block. default = 10m
blockTime = 10m

; How long to wait for authority nodes each factom-minute before faulting them. Should be higher
; than a tenth of "blockTime". default = 2m
faultTimeout = 2m

; How long an audit node has to volunteer before moving to the next one. default = 30s
roundTimeout = 30s

; Enable to force a node to always run as follower. default = false
forceFollower = false

; The chain id for the factoshi exchange rate updates.
; default = 111111118d918a8be684e0dac725493a75862ef96d2d3f43f84b26969329bf03
FERChain = 111111118d918a8be684e0dac725493a75862ef96d2d3f43f84b26969329bf03

; The public key that validates entries to the FER Chain.
; default = daf5815c2de603dbfa3e1e64f88a5cf06083307cf40da4a9b539c41832135b4a
FERPublicKey: daf5815c2de603dbfa3e1e64f88a5cf06083307cf40da4a9b539c41832135b4a

; The identity that signed the genesis block.
bootstrapIdentity = 38bab1455b7bd7e5efd15c53c777c79d0c988e9210f1da49a99d95b3a6417be9
bootstrapKey = cc1985cdfae4e32b5a454dfda8ce5e1361558482684f3367649c3ad852c8e31a

; Add balance hashes to ACKs. default = true
balanceHash = true

; Delay time for when to start processing messages. default = 0s
startDelay = 0s

; ---------------- IDENTITY ----------------
; The identity chain of this node. optional.
identityChain =

; The private key of the identity used to sign messages. Ed25519 key in hexadecimal.
identityPrivateKey = 4c38c72fc5cdad68f13b74674d3ffb1f3d63a112710868c9b08946553448d26d

; The public key of the identity used to sign messages. Ed25519 key in hexadecimal.
identityPublicKey = cc1985cdfae4e32b5a454dfda8ce5e1361558482684f3367649c3ad852c8e31a

; The height at which to activate the identity (for brainswaps). default = 0
identityActivationHeight = 0


; ---------------- WEB SERVICES ----------------
; The port at which to access the factomd API. default = 8088
apiPort = 8088

; The mode of operation of the control panel. default = READYONLY
; Choices are: DISABLED | READONLY | READWRITE
controlPanel = READONLY

; The web-port at which to access the control panel. default = 8090
controlPanelPort = 8090

; The display name of the node on the control panel.
controlPanelName = 

; If enabled, the pprof server will accept connections outside of localhost. default = false
pprofExpose = false

; Port for the pprof frontend. default = 6060
pprofPort = 6060

; Pprof memory profiling rate. 0 to disable, 1 for everything. default = 524288 (512kibi)
pprofMPR = 524288


; If TLS is enabled, the control panel and API will only be accessible via HTTPS. If you
; have a certificate, you can specify the location of the certificate and PEM key. 
; If you enable TLS without an existing certificate, factomd will generate a self-signed
; certificate inside HomeDir, using the specified addresses in addition to localhost
webTLS = false
webTLSKey =
webTLSCertificate =

; The addresses to include in the certificate. Repeat for every address.
webtlsAddress = 
; If set, the control panel and API will require basic http authentication to use
webUsername = 
webPassword = 
; This sets the Cross-Origin Resource Sharing (CORS) header for the API and Walletd.
; If left blank, CORS is disabled
webCORS = 


; ---------------- DATABASE ----------------
; Which database architecture to use
; Choice of LDB | BOLT | MAP
;   LDB: LevelDB (default)
;   BOLT: BoltDB
;   MAP: in-memory only database
dbType = LDB

; Set a unique identifier included in the path if you want run multiple databases in the same HomeDir
dbSlug = 

; Sub-path relative to HomeDir to store Ldb files
dbLdbPath = "database/ldb"

; Sub-path relative to HomeDir for BoltDB
dbBoltPath = "database/bolt"

; If enabled, factomd will turn on the block extractor to export blocks to disk
dbExportData = false

; Sub-path relative to HomeDir for exporting data
dbExportDataPath = "database/export/"

; Sub-path relative to HomeDir for the block extractor
dbDataStorePath = "data/export"

; Enable the use of the FastBoot file to cache block validation. default = true
dbFastBoot = true

; Create a FastBoot entry every X blocks. default = 1000
dbFastBootRate = 1000

; ---------------- P2P ----------------
; If disabled, the node will not connect to a network. default = true
p2pEnable = true

; If enabled, peers will be persisted to disk. default = true
p2pPeerFile = true

; The default ports used for network connections. default = 8108
p2pPort = 8108

; The URL of the seed file to use for bootstrapping.
p2pSeed =

; A list of peers that the node will always connect to in the format of "host:port". Repeatable.
; Example to add three special peers:
;   p2pSpecialPeer = "123.456.78.9:8108"
;   p2pSpecialPeer = "97.86.54.32:8108"
;   p2pSpecialPeer = "56.78.91.23:8108"
p2pSpecialPeer = 

; Which peers the node should allow.
; Choices:
;   NORMAL: allows all connections (default)
;   ACCEPT: the node accepts incoming connection but only dials to special peers
;   REFUSE: the node dials to special peers but refuses all incoming connections
p2pMode = NORMAL

; How long peers have to send or receive a message before timing out. default = 5m
p2pTimeout = 5m


; ---------------- LOGGING ----------------
; The level of messages to log. Higher levels are included. default = ERROR
; Choices (from lowest to highest level):
;   DEBUG | INFO | NOTICE | WARNING | ERROR | CRITICAL | ALERT | EMERGENCY | NONE
logLevel = ERROR

; The sub-path of HomeDir to store logs in
logPath = "database/Log"

; If enabled, log files will be written in JSON. default = false
logJson = false

; The URL of a logstash server to send logs to. Leave blank to disable
logLogstash = 

; Specify a file to write a copy of StdOut to file
logStdOut =

; Specify a file to write a copy of StdErr to file
logStdErr = 

; A regular expression of which message logs to save in the current working directory
; For more details see https://factomize.com/forums/threads/logging-in-factomd.1766/
logMessages = 

; Save DBStates to disk after being processed. default = false
; Files will be saved to dbLdbPath/<network>/dbstates/processed_dbstate_<height>.block
logDBStates = false


; ---------------- SIMULATION ----------------
; Enable console input. default = true
simConsole = true

; How many simulated nodes to launch with a minimum of one. default = 1
simCount = 1

; The node to focus on at startup. The first node starts at 0. default = 0
simFocus = 0

; The network structure of the simulated network. default = LONG
; Choices: FILE | SQUARE | LONG | LOOPS | ALOT | ALOT+ | TREE | CIRCLES
simNet = LONG

; The path to the sim node file for simNet=FILE
simNetFile = 

; Simulated drop rate for packets. Number of messages to drop out of 1000. default = 0
simDropRate = 0

; Time offset between clocks in simulated nodes. default = 0s
simTimeOffset = 0s

; If enabled, the node will keep track of recently sent messages that can be displayed
; in the console with the "m" command. default = false
simRuntimeLog = false

; Pause the processing of entries in the processlist. Equivalent to the "W" command. default = false
simWait = false


; ---------------- DEBUG ----------------
; The mode of the debug console.
; Choices:
;   OFF: no debug console (default)
;   LOCAL: only accepts connections from localhost and launches a terminal
;   ON: accepts remote connections
debugConsole = OFF

; The port to launch the console server. default = 8093
debugConsolePort = 8093

; If enabled, check the validity of chain heads on boot. default = true
chainHeadCheck = true

; If enabled, try to automatically fix any invalid chain heads. default = true
chainHeadFix = true

; If enabled, all entries for one factom-minute will be handled by a VM index 0
; instead of being distributed over all VMs. default = false
oneLeader = false

; Keep the node's DBState even if the signature doesn't match with the majority. default = false
keepMismatch = false

; Force the height on the second pass sync. Set to -1 to disable. default = -1
forceSync2Height = -1


; ---------------- JOURNALING ----------------
; Path to the journal file. Journaling disabled if left blank.
journalFile = 

; Whether to create a new journal or play back an existing journal default = READ
; Choices: CREATE | READ
journalMode = READ

; Force the node to run the journal as a specific node type.
; Choices:
;   AUTO: let node determine (default)
;   FOLLOWER: node is a follower
;   LEADER: node is a leader
journalType = AUTO


; ---------------- PLUGINS ----------------
; In order for plugins to be enabled, the binaries have to be located inside the plugin folder
; Path to the plugin binaries folder.
pluginPath = 

; Enable torrent sync plugin 
pluginTorrent = false

; If enabled, the node is an upload in the torrent network
pluginTorrentUpload = false


; ------------------------------------------------------------------------------
; Configurations for factom-walletd
; ------------------------------------------------------------------------------
[Walletd]
; These are the username and password that factom-walletd requires
; This file is also used by factom-cli to determine what login to use
WalletRpcUser = 
WalletRpcPass =

; These define if the connection to the wallet should be encrypted, and if it is, what files
; are the secret key and the public certificate.  factom-cli uses the certificate specified here if TLS is enabled.
; To use default files and paths leave /full/path/to/... in place.
WalletTlsEnabled                      = false
WalletTlsPrivateKey                   = "/full/path/to/walletAPIpriv.key"
WalletTlsPublicCert                   = "/full/path/to/walletAPIpub.cert"

; This is where factom-walletd and factom-cli will find factomd to interact with the blockchain
; This value can also be updated to authorize an external ip or domain name when factomd creates a TLS cert
FactomdLocation                       = "localhost:8088"

; This is where factom-cli will find factom-walletd to create Factoid and Entry Credit transactions
; This value can also be updated to authorize an external ip or domain name when factom-walletd creates a TLS cert
WalletdLocation                       = "localhost:8089"

; Enables wallet database encryption on factom-walletd. If this option is enabled, an unencrypted database
; cannot exist. If an unencrypted database exists, the wallet will exit.
WalletEncrypted                       = false


[factomd.MAIN]
FERPublicKey: daf5815c2de603dbfa3e1e64f88a5cf06083307cf40da4a9b539c41832135b4a
p2pPort: 8108
p2pSeed: https://raw.githubusercontent.com/FactomProject/factomproject.github.io/master/seed/mainseed.txt

[factomd.TEST]
p2pFERPublicKey: 1d75de249c2fc0384fb6701b30dc86b39dc72e5a47ba4f79ef250d39e21e7a4f
p2pPort: 8109
p2pSeed: https://raw.githubusercontent.com/FactomProject/factomproject.github.io/master/seed/testseed.txt

[factomd.LOCAL]
p2pFERPublicKey: 3b6a27bcceb6a42d62a3a8d02a6f0d73653215771de243a63ac048a18b59da29
p2pPort: 8110
p2pSeed: https://raw.githubusercontent.com/FactomProject/factomproject.github.io/master/seed/localseed.txt

[factomd.fct_community_test]
p2pFERPublicKey: 58cfccaa48a101742845df3cecde6a9f38037030842d34d0eaa76867904705ae
bootstrapIdentity: 8888882f5002ff95fce15d20ecb7e18ae6cc4d5849b372985d856b56e492ae0f
bootstrapKey: 58cfccaa48a101742845df3cecde6a9f38037030842d34d0eaa76867904705ae
p2pPort: 8110
p2pSeed: https://raw.githubusercontent.com/FactomProject/communitytestnet/master/seeds/testnetseeds.txt
```


# Implementation

The configuration code is placed in its own "config" package under `factomd/config`. To parse command line flags, the `go-flags` package is used as it provides built-in support for grouping, short/long flags, and enum types. To parse the config file, the `go-ini` package is used to improve malleability and improved group support (like subgroups).

The configuration itself is only specified in one location: the `settings.go` file, which contains . The default configuration and ini are generated

# Copyright

Copyright and related rights waived via
[CC0](https://creativecommons.org/publicdomain/zero/1.0/).