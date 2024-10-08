---
subcategory: "Metal"
---

# equinix_metal_vlan (Resource)

Provides a resource to allow users to manage Virtual Networks in their projects.

To learn more about Layer 2 networking in Equinix Metal, refer to

* https://deploy.equinix.com/developers/docs/metal/layer2-networking/overview/
* https://deploy.equinix.com/developers/docs/metal/layer2-networking/overview/#network-configuration-types

## Example Usage

```terraform
# Create a new VLAN in metro "esv"
resource "equinix_metal_vlan" "vlan1" {
  description = "VLAN in New Jersey"
  metro       = "sv"
  project_id  = local.project_id
  vxlan       = 1040
}
```

## Argument Reference

The following arguments are supported:

* `project_id` - (Required) ID of parent project.
* `metro` - (Optional) Metro in which to create the VLAN
* `facility` - (**Deprecated**) Facility where to create the VLAN. Use metro instead; read the [facility to metro migration guide](https://registry.terraform.io/providers/equinix/equinix/latest/docs/guides/migration_guide_facilities_to_metros_devices)
* `description` - (Optional) Description string.
* `vxlan` - (Optional) VLAN ID, must be unique in metro.

## Attributes Reference

In addition to all arguments above, the following attributes are exported:

* `id` - ID of the virtual network.

## Import

This resource can be imported using an existing VLAN ID (UUID):

```sh
terraform import equinix_metal_vlan {existing_vlan_id}
```
