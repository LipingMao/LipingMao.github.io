---
title: Kuryr libnetwork(2) -- basic api
---

## Libnetwork Remote Driver in Kuryr

### POST /Plugin.Activate

Handshake between Libnetwork and Kuryr, Kuryr will tell Libnetwork, it support NetworkDriver and IpamDriver, it will return {"Implements": ["NetworkDriver", "IpamDriver"]}


### POST /NetworkDriver.GetCapabilities

Kuryr will return the {'Scope': cfg.CONF.capability_scope}, capability_scope is define in configuration file, it would be local or global.


### POST /NetworkDriver.DiscoverNew

Kuryr did nothing now, just return success to libnetwork.


### POST /NetworkDriver.DiscoverDelete

Kuryr did nothing now, just return success to libnetwork.


### POST /NetworkDriver.CreateNetwork

Kuryr will create Neutron Network mapping with the container network.
```
if Existed Network:
    if Support Tag:
        Update Tag
    else:
        Update Network Name
else:
    if Support Tag:
        Create Network with Tag.
    else:
        Create Network with special Name.
If Subnetwork is not created:
    Create Subnetwork
```

### POST /NetworkDriver.DeleteNetwork

Kuryr will delete Neutron Network by id.


### POST /NetworkDriver.CreateEndpoint

Kuryr create or update the neutron port base on libnetwork request with IP, Kuryr will return ip and mac to libnetwork.


### POST /NetworkDriver.EndpointOperInfo

Kuryr return port status to libnetwork now.


### POST /NetworkDriver.DeleteEndpoint

Kuryr do nothing here now.


### POST /NetworkDriver.Join

Call kuryr lib binding.port_bind function to bind Neutron Port with Docker interface.


### POST /NetworkDriver.Leave

Call kuryr lib binding.port_unbind function to unbind Neutron Port with Docker interface.


### POST /NetworkDriver.ProgramExternalConnectivity

Config Security Group for container port.


### POST /NetworkDriver.RevokeExternalConnectivity

Remove Security Group for container port.


### POST /IpamDriver.GetCapabilities

IpamDriver get capabilities, return {'RequiresMACAddress': False}


### POST /IpamDriver.GetDefaultAddressSpaces

response the local and global address space: LocalDefaultAddressSpace and GlobalDefaultAddressSpace


### POST /IpamDriver.RequestPool

Kuryr create neutron subnetpool for libnetwork.


### POST /IpamDriver.RequestAddress

If port is gw, do nothing just return. If port is non gw, create neutron port for the address.


### POST /IpamDriver.ReleasePool

Kuryr will try to delete subnetpool.


### POST /IpamDriver.ReleaseAddress

Kuryr will delete all the port with this ip.
