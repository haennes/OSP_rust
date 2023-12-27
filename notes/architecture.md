# Architecture
- Launcher
	- Compiler
	- Made using Godot
		- fast dev speed
		- easy to develop, many APIs
		- no extraordinary things needed
- Game
	- Made using bevy
		- fast run speed
		- easy, safe to develop (rust)
		- extraordinary things needed
			- huge coordinates (space)
			- joining parts together
			- ECS needed as inheritance structures get confusing ([see here](https://github.com/GREEN-CHAOS/Spacecavate))
	- Consitst of
		- Client
			- [GUI (+ Plugins)](##-Client)
			- [Core](##-Core)
		- Server 
			- [Backend](##-Server)
			- [Core](##-Core)
- Plugins
- Files

## Core
Consists of:
- [Physics](###-Phyiscs)
- [Networking](###-Networking)
- [Saving Games](###-Saves)
- [Parts (for example tank, engine....)](###-Components)
- [Plugins (client and server side needed)](###-Plugins)
	- integration of 3rd-Party chat
### Physics
Consists of:
-[Gravity](####-Gravity)
-[Newtons 3rd law](####-Newtons-3rd-law)
-[Collision](####-Collision)
#### Gravity
Gravity will be implemented using the non-relativistic formula.
This system gets all entities with a [mass]() and a [transform]() component and adds the force calculated divided by the mass to the acceleration field of the transform.
#### Newtons 3rd law
Newtons 3rd law states: To every action there is a equal and opposite reaction. This is how [Engines]() and [RCSThrusters]() work and thats why it is essential for this game. Effects of this might also appear when undocking small ships
#### Collision
Implementation of this will happen a little bit later in the development stage and will preferably use a library (open to suggestions (ideally already implemented for the bevyengine))
### Networking
Consists of:
- [When is data sent?](####-Send-Data-Events)
- [What data is sent?](####-Content)
- [Data format](####-Serialisation)
- [More efficient serializations (future)](###-Future-features)
#### Send Data Events
Bandwith should not be wasted in any way. Thats why the server should only send data when its neccesary
The following events lead to the server sending data to the client (A):
- Every 500s (is impemented as const, args) [all data](#####-State) is sent
- A changes any state -> broadcast changes to all except A
- Client asks for state upon joining, when it has conflicting data (**spam protection needed**)
- permissions change for A
- Plugin sents data
#### Content
This specifies the Content for the different Packages
The folowing Packages are specified.
- [State](#####-State)
- [Change](#####-Change)
- [Request](#####-Request)
- [Hello](#####-Hello)
- [Version(s)](#####-Versions)
- [Initialize to Server](#####-Init-Server)
- [Initialize to Client](#####-Init-Client)
##### State
Sends the entire current State with the universe time when the snapshot was taken. Sending this should be avoided as best as possible as this needs a **LOT** of bandwith. This package should only be sent in the login stage. To avoid small errors this package will also be sent every 500s
**Specification**
```rust
State{
	time: u64,
	entities: Vec<Entity>
}

```
##### Change
This package should be used to communicate any changes happening in the universe. As this Package is very small it can be used extensively.
**THIS NEEDS TO BE APPLIED TO HOW BEVY WORKS**
**Specification**
```rust
Change{
	time: u64
	Component: Component
}
```

##### Request
This package is used to request certain information. This should not be needed often. 
**Specification**
```rust
	enum RequestType{
		SET(value: Bytes), 	//Set a specific field
		GET(), 			//Get value of specific field
		HAS(try_create: bool)	//Check wether a speciific field exists
	}
	struct Request{
		kind: RequestType,
		property: String //maybe change to int??
}
```
##### Hello
This package is the first package that is sent from the client to the Server. It only content is the clients IP-address. The server answers this package with the [Versions][#####-Versions] package
**The format of this package may never be changed**
**Specification**
```rust
	struct Hello{
		ip: IpAddress
}
```
##### Versions
This package specifies which versions of the game are supported by the server. The listing of the Versions goes form most preferred to available / deprecated
**The format of this package may never be changed**
**Specification**
```rust
struct Versions{
	versions: Vec<Version>
}	
```
##### ChosenVersion
This package specifies which verison the client chose and is the answer to the [Versions](#####-Versions) package.
**The format of this package may never be changed**
**Specification**
```rust
struct ChosenVersion{
	version: Version
}
```
##### Init Server
This package is sent from the Client to the Server to initialize the communication
**Specification**
```rust
	InitServer{
		id: ID
		pub_key: u256
		IpAddr : IpAddress
		character: Character
		available_plugins: Vec<Plugin>
}
```
##### Init Client
This package is  sent form the Server to the Client as an answert to [Init Server](#####-Init-Server)
**Specification**
```rust
	InitClient{
		pub_key: u256,
		State: State, //Running, Maintainance, ...
		accepted: bool,
		needed_plugins: Vec<Plugin>,
		provideable_plugins: Vec<Plugin> // needed_plugins - available_plugins that can be downloaded from the server
}
```
#### Serialisation
For this the serde crate will be used. Into what format the data is serialized is to be decided
#### Future features
future.
### Saves
Consists of:
-[Universe](####-Universe)
-[Plugins](####-Plugins)
-[Implementing rollback features](####-Git)
#### Universe
For saving games all enties are sorted and then saved in RON in a file located at {SAVES_PATH}/{WORLD_NAME}/universe.ron
#### Plugins
For plugins that need to save configurations or other non-entity-related data there is a storage API provided to save them at:
- {SAVES_PATH}/{WORLD_NAME}/{PLUGIN_NAME}
- {PLUGINS_PATH}/{PLUGIN_NAME}
#### Git
To be able to quickly and easily revert changes made by accident or to make it possible to uninstall a plugin safely.
For this to work every plugin is a own git user with the user.name being the name of the plugin and the user.email being the email of the maintainer of the plugin.
The user should have options to decide how accurately the changes shall be tracked:
- [Fast](####-Fast)
- [Medium](####-Medium)
- [Off](####-Off)
#### Fast
This creates a commit for each access of the storage API
#### Medium
This creates a commit for each time the game is saved. The changes in the non-universe-specific directory are commited on exit of the game
#### Off
This deactivates git completely resulting in not being able to roll back

### Components
Consists of:
- [Parts](####-Parts)
- [Components](####-Components)
#### Parts
All the rocket parts that are needed:
#### Components
All the components that are needed:

### Plugins
A Plugin is an extension to the functionality of the game. But it hast to be precompiled. If a server requires a plugin it will tell the client the version and the name of the plugin. The acutal game will stop after sending the launcher the request to install the required plugins. After the plugins are installed and the game is compiled it will be restarted. This way the game stays fast and installing Plugins is easy.

## Client
Consists of:
- [User Interface](###-UI)
- [Client only plugins](###-Client-only-plugins)
	- mapscreen
	- overlays
	- web interface
### UI
The UI should be designed using bevys UI system. Some options could be exposed to a webui accessible through a browser.
Bevy supports multiple windows, which should be added in the future
### Client only plugins
Plugins only neede at the client side. This could be a mapintegration, proximitychat with a 3rd-party chatservice
## Server
Consists of:
- [Communication](###-Communication)
- [Server only plugins](###-Server-only-plugins)
	- Auto-Backup
	- web interface
### Communication
The server needs to know what has to be sent at which time.

### Server only plugins
Plugins only needed on the server
In the future a  webserver or a backupsystem might be shipped with the game
