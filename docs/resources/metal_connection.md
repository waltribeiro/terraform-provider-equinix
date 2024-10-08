---
subcategory: "Metal"
---

# equinix_metal_connection (Resource)

Use this resource to request the creation an Interconnection asset to connect with other parties using [Equinix Fabric - software-defined interconnections](https://deploy.equinix.com/developers/docs/metal/interconnections/introduction/).

~> Equinix Metal connection with Service Token A-side / Z-side (service_token_type) is not generally available and may not be enabled yet for your organization.

## Example Usage

### Shared Connection with a_side token - Redundant Connection from Equinix Metal to a Cloud Service Provider

```terraform
resource "equinix_metal_connection" "example" {
    name               = "tf-metal-to-azure"
    project_id         = local.project_id
    type               = "shared"
    redundancy         = "redundant"
    metro              = "sv"
    speed              = "1000Mbps"
    service_token_type = "a_side"
    contact_email      = "username@example.com"
}

data "equinix_fabric_sellerprofile" "example" {
  name                     = "Azure ExpressRoute"
  organization_global_name = "Microsoft"
}

resource "equinix_fabric_connection" "example" {
  name              = "tf-metal-to-azure"
  profile_uuid      = data.equinix_fabric_sellerprofile.example.uuid
  speed             = azurerm_express_route_circuit.example.bandwidth_in_mbps
  speed_unit        = "MB"
  notifications     = ["example@equinix.com"]
  service_token     = equinix_metal_connection.example.service_tokens.0.id
  seller_metro_code = "AM"
  authorization_key = azurerm_express_route_circuit.example.service_key
  named_tag         = "PRIVATE"
  secondary_connection {
    name          = "tf-metal-to-azure-sec"
    service_token = equinix_metal_connection.example.service_tokens.1.id
  }
}
```

-> NOTE: There are multiple [Equinix Fabric L2 Connection Terraform modules](https://registry.terraform.io/search/modules?namespace=equinix-labs&q=fabric-connection) available with full-fledged examples of connections from Fabric Ports, Network Edge Devices or Service Token to most popular Cloud Service Providers. Check out the examples for Equinix Metal shared connection with A-side Service Token included in each of them: [AWS](https://registry.terraform.io/modules/equinix-labs/fabric-connection-aws/equinix/latest/examples/service-token-metal-to-aws-connection), [Azure](https://registry.terraform.io/modules/equinix-labs/fabric-connection-azure/equinix/latest/examples/service-token-metal-to-azure-connection), [Google Cloud](https://registry.terraform.io/modules/equinix-labs/fabric-connection-gcp/equinix/latest/examples/service-token-metal-to-gcp-connection), [IBM Cloud](https://registry.terraform.io/modules/equinix-labs/fabric-connection-ibm/equinix/latest/examples/service-token-metal-to-ibm-connection), [Oracle Cloud](https://registry.terraform.io/modules/equinix-labs/fabric-connection-oci/equinix/latest/examples/service-token-metal-to-oci-connection), [Alibaba Cloud](https://registry.terraform.io/modules/equinix-labs/fabric-connection-alibaba/equinix/latest/examples/service-token-metal-to-alibaba-connection).

### Shared Connection with z_side token - Non-redundant Connection from your own Equinix Fabric Port to Equinix Metal

```terraform
resource "equinix_metal_vlan" "example" {
    project_id      = local.my_project_id
    metro           = "FR"
}

resource "equinix_metal_connection" "example" {
    name               = "tf-port-to-metal"
    project_id         = local.project_id
    type               = "shared"
    redundancy         = "primary"
    metro              = "FR"
    speed              = "200Mbps"
    service_token_type = "z_side"
    contact_email      = "username@example.com"
    vlans              = [
      equinix_metal_vlan.example.vxlan
    ]
}

data "equinix_fabric_port" "example" {
  name = "CX-FR5-NL-Dot1q-BO-1G-PRI"
}

resource "equinix_fabric_connection" "example" {
  name                = "tf-port-to-metal"
  zside_service_token = equinix_metal_connection.example.service_tokens.0.id
  speed               = "200"
  speed_unit          = "MB"
  notifications       = ["example@equinix.com"]
  port_uuid           = data.equinix_fabric_port.example.id
  vlan_stag           = 1020
}
```

-> NOTE: There is an [Equinix Fabric L2 Connection To Equinix Metal Terraform module](https://registry.terraform.io/modules/equinix-labs/fabric-connection-metal/equinix/latest) available with full-fledged examples of connections from Fabric Ports, Network Edge Devices or Service Tokens. Check out the [example for shared connection with Z-side Service Token](https://registry.terraform.io/modules/equinix-labs/fabric-connection-metal/equinix/0.2.0/examples/fabric-port-connection-with-zside-token).

### Shared Connection for organizations without Connection Services Token feature enabled

```terraform
resource "equinix_metal_vlan" "example1" {
    project_id      = local.my_project_id
    metro           = "SV"
}

resource "equinix_metal_vlan" "example2" {
    project_id      = local.my_project_id
    metro           = "SV"
}

resource "equinix_metal_connection" "example" {
    name            = "tf-port-to-metal-legacy"
    project_id      = local.my_project_id
    metro           = "SV"
    redundancy      = "redundant"
    type            = "shared"
    contact_email   = "username@example.com"
    vlans              = [
      equinix_metal_vlan.example1.vxlan,
      equinix_metal_vlan.example2.vxlan
    ]
}

data "equinix_fabric_port" "example" {
  name = "CX-FR5-NL-Dot1q-BO-1G-PRI"
}

resource "equinix_fabric_connection" "example" {
  name                = "tf-port-to-metal-legacy"
  speed               = "200"
  speed_unit          = "MB"
  notifications       = ["example@equinix.com"]
  port_uuid           = data.equinix_fabric_port.example.id
  vlan_stag           = 1020
  authorization_key   = equinix_metal_connection.example.token
}
```

### Shared Connection with authorization_code Non-redundant Metal-Fabric Integration connection from Equinix Metal to a Cloud Service Provider via Equinix Fabric Port

```terraform
resource "equinix_metal_vlan" "example1" {
  project_id      = local.my_project_id
  metro           = "SV"
}
resource "equinix_metal_connection" "example" {
  name            = "tf-port-to-metal-legacy"
  project_id      = local.my_project_id
  metro           = "SV"
  redundancy      = "primary"
  type            = "shared_port_vlan"
  contact_email   = "username@example.com"
  vlans              = [ equinix_metal_vlan.example1.vxlan ]
}
data "equinix_fabric_service_profiles" "zside" {
  filter {
    property = "/name"
    operator = "="
    values   = ["AWS Direct Connect"]
  }
}
resource "equinix_fabric_connection" "example" {
  name = "tf-NIMF-metal-2-aws-legacy"
  type = "EVPL_VC"
  notifications {
    type   = "ALL"
    emails = "sername@example.com"
  }
  project {
    project_id = local.fabric_project_id
  }
  bandwidth       = "200"
  order {
    purchase_order_number = "1-323292"
  }
  a_side {
    access_point {
      type               = "METAL_NETWORK"
      authentication_key = equinix_metal_connection.example.authorization_code
    }
  }
  z_side {
    access_point {
      type               = "SP"
      authentication_key = local.aws_account_id
      seller_region      = "us-west-1"
      profile {
        type = "L2_PROFILE"
        uuid = data.equinix_fabric_service_profiles.zside.id
      }
      location {
        metro_code ="SV"
      }
    }
  }
}
```

### Shared Connection with authorization_code Non-redundant Metal-Fabric Integration connection from Equinix Fabric Cloud Router to Equinix Metal

```terraform
resource "equinix_metal_vlan" "example1" {
  project_id      = local.my_project_id
  metro           = "SV"
}
resource "equinix_metal_connection" "example" {
  name            = "tf-port-to-metal-legacy"
  project_id      = local.my_project_id
  metro           = "SV"
  redundancy      = "primary"
  type            = "shared_port_vlan"
  contact_email   = "username@example.com"
  vlans              = [ equinix_metal_vlan.example1.vxlan ]
}
resource "equinix_fabric_connection" "example" {
  name = "tf-NIMF-metal-2-aws-legacy"
  type = "IP_VC"
  notifications {
    type   = "ALL"
    emails = "sername@example.com"
  }
  project {
    project_id = local.fabric_project_id
  }
  bandwidth       = "200"
  order {
    purchase_order_number = "1-323292"
  }
  a_side {
      access_point {
        type = "CLOUD_ROUTER"
        router {
          uuid = local.cloud_router_uuid
        }
      }
  }
  z_side {
    access_point {
      type               = "METAL_NETWORK"
      authentication_key = equinix_metal_connection.example.authorization_code
    }
  }
}
```

## Argument Reference

The following arguments are supported:

* `name` - (Required) Name of the connection resource
* `metro` - (Optional) Metro where the connection will be created.
* `facility` - (**Deprecated**) Facility where the connection will be created. Use metro instead; read the [facility to metro migration guide](https://registry.terraform.io/providers/equinix/equinix/latest/docs/guides/migration_guide_facilities_to_metros_devices)
* `redundancy` - (Required) Connection redundancy - redundant or primary.
* `type` - (Required) Connection type - dedicated or shared.
* `contact_email` - (Optional) The preferred email used for communication and notifications about the Equinix Fabric interconnection. Required when using a Project API key. Optional and defaults to the primary user email address when using a User API key.
* `project_id` - (Optional) ID of the project where the connection is scoped to, must be set for.
* `speed` - (Required) Connection speed - Values must be in the format '<number>Mbps' or '<number>Gpbs', for example '100Mbps' or '50Gbps'. Actual supported values will depend on the connection type and whether the connection uses VLANs or VRF.
* `description` - (Optional) Description for the connection resource.
* `mode` - (Optional) Mode for connections in IBX facilities with the dedicated type - standard or tunnel. Default is standard.
* `tags` - (Optional) String list of tags.
* `vlans` - (Optional) Only used with shared connection. Vlans to attach. Pass one vlan for Primary/Single connection and two vlans for Redundant connection.
* `service_token_type` - (Optional) Only used with shared connection. Type of service token to use for the connection, a_side or z_side. (**NOTE: To support the legacy non-automated way to create connections, terraform will not check if `service_token_type` is specified. If your organization already has `service_token_type` enabled, be sure to specify it or the connection will return a legacy connection token instead of a service token**)

## Attributes Reference

In addition to all arguments above, the following attributes are exported:

* `organization_id` - ID of the organization where the connection is scoped to.
* `status` - Status of the connection resource.
* `ports` - List of connection ports - primary (`ports[0]`) and secondary (`ports[1]`). Schema of port is described in documentation of the [equinix_metal_connection datasource](../data-sources/metal_connection.md).
* `service_tokens` - List of connection service tokens with attributes required to configure the connection in Equinix Fabric with the [equinix_fabric_connection](./fabric_connection.md) resource or from the [Equinix Fabric Portal](https://fabric.equinix.com/dashboard). Scehma of service_token is described in documentation of the [equinix_metal_connection datasource](../data-sources/metal_connection.md).
* `token` - (Deprecated) Fabric Token required to configure the connection in Equinix Fabric with the [equinix_fabric_connection](./fabric_connection.md) resource or from the [Equinix Fabric Portal](https://fabric.equinix.com/dashboard). If your organization already has connection service tokens enabled, use `service_tokens` instead.
* `authorization_code` - Fabric Authorization code to configure the Metal-Fabric Integration connection with Cloud Service Provider through Equinix Fabric with the [equinix_fabric_connection](./fabric_connection.md) resource from the [Equinix Developer Portal](https://developer.equinix.com/dev-docs/fabric/getting-started/fabric-v4-apis/connect-metal-to-amazon-web-services).
