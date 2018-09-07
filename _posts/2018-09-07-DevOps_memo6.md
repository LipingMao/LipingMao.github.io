---
title: OpenStack  -- Nova Ocata Specs
---

## Background

> Go through [Nova Ocata version specs](http://specs.openstack.org/openstack/nova-specs/specs/ocata/index.html)


## [Add swap volume notifications](http://specs.openstack.org/openstack/nova-specs/specs/ocata/implemented/add-swap-volume-notifications.html)

Add the following notifications.
```
    instance.volume_swap.start
    instance.volume_swap.end
    instance.volume_swap.error
```

## [Add whitelist for filter and sort query parameters for server list API](http://specs.openstack.org/openstack/nova-specs/specs/ocata/implemented/add-whitelist-for-server-list-filter-sort-parameters.html)

add a strict set of whitelisted filtering and sorting parameters for server list/detail operations.

## [Consistent Query Parameters Validation](http://specs.openstack.org/openstack/nova-specs/specs/ocata/implemented/consistent-query-parameters-validation.html)

propose one consistent mechanism to validate query parameters.

## [Resource Providers - Custom Resource Classes](http://specs.openstack.org/openstack/nova-specs/specs/ocata/implemented/custom-resource-classes.html)

propose to provide the ability for an administrator to create a set of special resource classes that indicate deployer-specific resources that can be provided by a resource provider.

## [Deprecated image-metadata proxy API](http://specs.openstack.org/openstack/nova-specs/specs/ocata/implemented/deprecate-image-meta-proxy-api.html)

The proxies APIs were deprecated in the propose of Deprecated API Proxies. But the image-metadata API missed in that propose. This spec aims to describe the deprecation of image-metadata API.

## [Notifications on flavor operations](http://specs.openstack.org/openstack/nova-specs/specs/ocata/implemented/flavor-notifications.html)

Versioned notifications will be added for the following actions:
```
    Flavor.create
    Flavor.save_projects
    Flavor.add_access
    Flavor.remove_access
    Flavor.save_extra_specs
    Flavor.destroy
```

## [Ironic virt driver: static portgroups support](http://specs.openstack.org/openstack/nova-specs/specs/ocata/implemented/ironic-portgroups-support.html)

To allow the utilization of NIC aggregation when instance is spawned on hardware server. Bonded NICs should be picked with higher preference than single NICs. It will allow user to increase performance or provide higher reliability of network connection.

## [Libvirt: Use the virtlogd deamon](http://specs.openstack.org/openstack/nova-specs/specs/ocata/implemented/libvirt-virtlogd.html)

If the serial console feature is enabled on a compute node with [serial_console].enabled = True it deactivates the logging of the boot messages. From a REST API perspective, this means that the two APIs os-getConsoleOutput and os-getSerialConsole are mutually exclusive. Both APIs can be valuable for cloud operators in the case when something goes wrong during the launch of an instance. This blueprint wants to lift the XOR relationship between those two REST APIs.

## [Simple tenant usage pagination](http://specs.openstack.org/openstack/nova-specs/specs/ocata/implemented/paginate-simple-tenant-usage.html)

The blueprint aims to add optional limit and marker parameters to the GET /os-simple-tenant-usage endpoints.

## [PCI Pass-through Whitelist Regex](http://specs.openstack.org/openstack/nova-specs/specs/ocata/implemented/pci-passthrough-whitelist-regex.html)

Enhance PCI pass-through whitelist to support regular expression for address attributes.

## [Resource Providers - Filtered Resource Providers by Request](http://specs.openstack.org/openstack/nova-specs/specs/ocata/implemented/resource-providers-get-by-request.html)

This blueprint aims to modify the POST method of the resource_providers REST resource so that it returns a list of resource providers that can support the list of requested resource classes.

## [Resource Providers - Scheduler Filters in DB](http://specs.openstack.org/openstack/nova-specs/specs/ocata/implemented/resource-providers-scheduler-db-filters.html)

This blueprint aims to have the scheduler calling the placement API for getting the list of resource providers that could allow to pre-filter compute nodes from evaluation during select_destinations().

## [http://specs.openstack.org/openstack/nova-specs/specs/ocata/implemented/return-uuid-from-os-aggregates-api.html](http://specs.openstack.org/openstack/nova-specs/specs/ocata/implemented/return-uuid-from-os-aggregates-api.html)

This spec proposes that the os-aggregates REST API returns the aggregate UUID in a new microversion so that the aggregate UUID can be used to associate an aggregate with resource providers in the Placement service.

## [Expose SR-IOV physical function’s VLAN tag to guests](http://specs.openstack.org/openstack/nova-specs/specs/ocata/implemented/sriov-pf-passthrough-neutron-port-vlan.html)

The sriov-pf-passthrough-neutron-port spec [1](https://review.openstack.org/#/c/239875/), that introduced network awareness for the passed-through Physical Functions, has been implemented in the Newton Cycle. However, current implementation ignores VLAN tags set on the associated neutron port.

## [Use service token for long running tasks](http://specs.openstack.org/openstack/nova-specs/specs/ocata/implemented/use-service-tokens.html)

Make use of new Keystone feature where if service token is sent along with the user token,then it will ignore the expiration of user token. It stop issues with user tokens expiring during long running operations, such as live-migration.

## [vendordata reboot, remaining work in Ocata](http://specs.openstack.org/openstack/nova-specs/specs/ocata/implemented/vendordata-reboot-ocata.html)

In Newton, the Nova team implemented a new way for deployers to provide vendordata to instances via configdrive and metadata server. There were a few smaller items that didn’t land in Newton, which we need to finalize.
