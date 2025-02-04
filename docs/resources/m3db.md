# M3 DB Resource

The M3 DB resource allows the creation and management of Aiven M3 services.

## Example Usage

```hcl
resource "aiven_m3db" "m3" {
    project = data.aiven_project.foo.project
    cloud_name = "google-europe-west1"
    plan = "business-8"
    service_name = "my-m3db"
    maintenance_window_dow = "monday"
    maintenance_window_time = "10:00:00"
    
    m3db_user_config {
      m3db_version = 0.15
                    
        namespaces {
            name = "my-ns1"
            type = "unaggregated"
        }
    }
}
```

## Argument Reference

* `project` - (Required) identifies the project the service belongs to. To set up proper dependency
between the project and the service, refer to the project as shown in the above example.
Project cannot be changed later without destroying and re-creating the service.

* `service_name` - (Required) specifies the actual name of the service. The name cannot be changed
later without destroying and re-creating the service so name should be picked based on
intended service usage rather than current attributes.

* `cloud_name` - (Optional) defines where the cloud provider and region where the service is hosted
in. This can be changed freely after service is created. Changing the value will trigger
a potentially lengthy migration process for the service. Format is cloud provider name
(`aws`, `azure`, `do` `google`, `upcloud`, etc.), dash, and the cloud provider
specific region name. These are documented on each Cloud provider's own support articles,
like [here for Google](https://cloud.google.com/compute/docs/regions-zones/) and
[here for AWS](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.RegionsAndAvailabilityZones.html).

* `plan` - (Optional) defines what kind of computing resources are allocated for the service. It can
be changed after creation, though there are some restrictions when going to a smaller
plan such as the new plan must have sufficient amount of disk space to store all current
data and switching to a plan with fewer nodes might not be supported. The basic plan
names are `hobbyist`, `startup-x`, `business-x` and `premium-x` where `x` is
(roughly) the amount of memory on each node (also other attributes like number of CPUs
and amount of disk space varies but naming is based on memory). The exact options can be
seen from the Aiven web console's Create Service dialog.

* `project_vpc_id` - (Optional) optionally specifies the VPC the service should run in. If the value
is not set the service is not run inside a VPC. When set, the value should be given as a
reference as shown above to set up dependencies correctly and the VPC must be in the same
cloud and region as the service itself. Project can be freely moved to and from VPC after
creation but doing so triggers migration to new servers so the operation can take
significant amount of time to complete if the service has a lot of data.

* `termination_protection` - (Optional) prevents the service from being deleted. It is recommended to
set this to `true` for all production services to prevent unintentional service
deletion. This does not shield against deleting databases or topics but for services
with backups much of the content can at least be restored from backup in case accidental
deletion is done.

* `maintenance_window_dow` - (Optional) day of week when maintenance operations should be performed. 
On monday, tuesday, wednesday, etc.

* `maintenance_window_time` - (Optional) time of day when maintenance operations should be performed. 
UTC time in HH:mm:ss format.

* `m3db_user_config` - (Optional) defines M3 specific additional configuration options. The following 
configuration options available:
    * `m3db_version` - (Optional) M3 major version
    * `custom_domain` - (Optional) Serve the web frontend using a custom CNAME pointing to the Aiven DNS name.
    * `ip_filter` - (Optional) Allow incoming connections from CIDR address block, e.g. '10.20.0.0/16'
    * `project_to_fork_from` - (Optional) Name of another project to fork a service from. This has
    effect only when a new service is being created.
    * `service_to_fork_from` - (Optional)  Name of another service to fork from. This has effect only 
    when a new service is being created.
    * `m3coordinator_enable_graphite_carbon_ingest` - (Optional)  Enables access to Graphite Carbon 
    plaintext metrics ingestion. It can be enabled only for services inside VPCs. The 
    metrics are written to aggregated namespaces only.
    * `private_access` - (Optional) Allow access to selected service ports from private networks.
        * `m3coordinator` - (Optional) Allow clients to connect to m3coordinator with a DNS name that 
        always resolves to the service's private IP addresses. Only available in certain network locations.
    * `public_access` - (Optional) Allow access to selected service ports from the public Internet.
        * `m3coordinator` - (Optional) Allow clients to connect to m3coordinator from the public internet 
        for service nodes that are in a project VPC or another type of private network.
    * `rules` - (Optional) Mapping rules allow more granular use of aggregation, not simply sending 
    everything to a namespace. If mapping rules exist that target a namespace, only data matching mapping 
    rules will be sent to it and nothing else.
        * `aggregations` - (Optional) List of aggregations to be applied
        * `drop` - (Optional) Drop the matching metric; Only store the derived metric (as specified in the roll-up rules), if any.
        * `filter` - (Optional) The metrics to be used with this particular rule; Matching metric names with wildcards (using
        __name__:wildcard) or matching tags and their (optionally wildcarded) values. For value, !
        can be used at start of value for negation, and multiple filters can be supplied using space as separator.
        * `name` - (Optional) The name of the rule.
        * `namespaces` - (Optional) Namespaces matching this storage policy; This rule will be used to store 
        the metrics in the given namespace(s). If a namespace is target of rules, the global default 
        aggregation will be automatically disabled. Note that specifying filters that match no namespaces 
        whatsoever will be returned as an error.
            * `oneOf` - (Optional) Namespaces matching this storage policy
        * `tags` - (Optional) List of tags to be appended to matching metrics.
            * `name` - (Optional) Name of the tag.
            * `value` - (Optional) Value of the tag.
    * `limits` - (Optional) M3 limits
        * `global_datapoints` - (Optional) The maximum number of data points fetched during request
        * `query_datapoints` - (Optional) The maximum number of data points fetched in single query
        * `query_require_exhaustive` - (Optional) When query limits are exceeded, whether to return error 
        (if True) or return partial results (False)
        * `query_series` - (Optional) The maximum number of series fetched in single query
    * `namespaces` - (Optional) List of M3 namespaces
        * `name` - (Required) The name of the namespace
        * `type` - (Required) The type of aggregation (aggregated/unaggregated)
        * `resolution` - (Optional) The resolution for an aggregated namespace
        * `options` - (Optional) Namespace options
            * `snapshot_enabled` - (Optional) Controls whether M3DB will create snapshot files for 
            this namespace
            * `writes_to_commitlog` - (Optional) Controls whether M3DB will include writes to this 
            namespace in the commitlog.
            * `retention_options` - (Optional) Retention options
                * `block_data_expiry_duration` - (Optional) Controls how long we wait before expiring stale data
                * `blocksize_duration` - (Optional) Controls how long to keep a block in memory before 
                flushing to a fileset on disk
                * `buffer_future_duration` - (Optional) Controls how far into the future writes to 
                the namespace will be accepted
                * `buffer_past_duration` - (Optional) Controls how far into the past writes to the 
                namespace will be accepted
                * `retention_period_duration` - (Optional) Controls the duration of time that M3DB will 
                retain data for the namespace

* `timeouts` - (Optional) a custom client timeouts.
    
## Attribute Reference

In addition to all arguments above, the following attributes are exported:

* `service_uri` - URI for connecting to the M3 service.

* `service_host` - M3 hostname.

* `service_port` - M3 port.

* `service_password` - Password used for connecting to the M3 service, if applicable.

* `service_username` - Username used for connecting to the M3 service, if applicable.

* `state` - Service state.

* `m3db` - M3 specific server provided values.

Aiven ID format when importing existing resource: `<project_name>/<service_name>`, where `project_name`
is the name of the project, and `service_name` is the name of the M3 service.
