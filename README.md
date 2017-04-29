# OpenNebula NetApp Drivers

These drivers are based on the OpenNebula shared drivers with the following differences:

* Instead of executing a regular `cp` in the hypervisor, it will ssh to the NetApp and execute `volume file clone create ...`

**TODO:**

* Verify the RESIZE feature.
* Use NetApp operation for disk snapshots

### Checkout the code

    git clone https://github.com/OpenNebula/addon-netapp

### Installation

Install by running `./install.sh`:

    cd addon-netapp
    sudo ./install.sh -u oneadmin -g oneadmin

### OpenNebula Configuration

Add `netapp` to the list of supported TMs in the `TM_MAD` directive in `/etc/one/oned.conf`.

    TM_MAD = [
        EXECUTABLE = "one_tm",
        ARGUMENTS = "-t 15 -d ...,netapp"
    ]

Now add the specific netapp configuration:

    TM_MAD_CONF = [
        NAME = "netapp", LN_TARGET = "NONE", CLONE_TARGET = "SYSTEM", SHARED = "YES",
        DS_MIGRATE = "YES"
    ]

Finally you need to add these parameters to the INHERIT section:

    INHERIT_DATASTORE_ATTR  = "NETAPP_HOST"
    INHERIT_DATASTORE_ATTR  = "NETAPP_VOLUME"
    INHERIT_DATASTORE_ATTR  = "NETAPP_PATH"

You will need to restart OpenNebula after this change.

### Configure image datastore

You will need to add the following parameters to your image datastore template:

* `NETAPP_HOST`: The frontend will ssh from the oneadmin account to this host. Make sure the passwordless ssh is configured.
* `NETAPP_VOLUME`: The name of the NetApp volumen on which it will operate through ssh.
* `NETAPP_PATH`: The path of `/var/lib/one/datastore` as seen from the NetApp.
* `TM_MAD`: It must be `netapp`

In order to apply these changes update the datastore's template. For example:

    $ onedatastore update <ds_id>
    ...
    TM_MAD="netapp"
    NETAPP_HOST="my-netapp.local"
    NETAPP_VOLUME="openvm_storage"
    NETAPP_PATH="/datastores"

### Configure system datastore

Make sure that your system datastore is using `TM_MAD="shared"`.

